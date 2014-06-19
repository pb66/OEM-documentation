## Fixed interval

In many if not most applications time series data is recorded at a fixed interval. A temperature or power measurement is made every 10 seconds, minute or hour. Given this highly regular nature of the timeseries data we can do away with the need to record every datapoint's timestamp and instead just record the start time of the timeseries and the timeseries interval in a meta file and then only record the datapoint values in the datafile. The timestamp for any given datapoint can easily be worked out by the start time, interval and the position of the data point in the file.

There are two main advantage of this approach versus the variable interval approach. 

1. The first advantage is that if we want to read a datapoint at a given time we dont need to search for the datapoint as we can calculate its position in the file directly. This reduces  the amount of reads required to fetch a datapoint from up to 30 reads down to 1 giving a significant performance improvement.

2. The second advantage is that in a timeseries where the data is highly reliable the disk use can be up to half that of the disk use used by a variable interval engine, due to not needing to record a timestamp for each datapoint.

The disadvantage is that when you have a gap in your data, that gap needs to be padded with NAN values, those times need to be allocated even if not used. This isnt a big a disadvantage as it may first seem because even if you had a gap of 6 months out of a year you would still only consume as much disk space as the variable interval method. Another perhaps more significant disadvantage is that you need to set the interval of the feed when you create the feed.

Two files are required for each fixed interval feed, a meta file to store the start time and interval and the datafile to store the 4 byte data values:

**Metafile:**

**Datafile:**

![Fixed Interval data file structure](files/fixedinterval.png)

### Writing to the timeseries data file

To write fixed interval data there are three steps:

1) Calculate the file position of the datapoint to be written.

    $timestamp = floor($timestamp / $meta->interval) * $meta->interval;
    $position = floor(($timestamp - $meta->start_time) / $meta->interval);

2) If there is a gap between the datapoint to be written and the last datapoint then padd the gap with NAN values, to padd efficiently buffer the padding.

    $buffer = pack("f",NAN);
    $buffer .= …
    fwrite($fh,$buffer);

3) Write the new datapoint at the end.

    fseek($fh,4*$position);
    fwrite($fh,pack("f",$value));

### Reading from the timeseries data file.

The get data query parameters are the start time, end time and the data interval of the output data.

1) Find the position of the datapoint nearest the query start time and calculate the skip size (number of datapoints we need to skip) in order to output the datapoints at the requested data interval.

    $start\_position = ceil(($query\_start - $meta->start_time) / $meta->interval);
    $skip\_size = round($out\_interval / $meta->interval);

2) Itterate through the data file from the start position reading data points at the skip size until the end time or the end of the file is reached.

    $data = array();
    $fh = fopen($this->dir.$meta->id.".dat", 'rb');
    while($time<=$end)
    {
        // $position steps forward by skipsize every loop
        $pos = ($startpos + ($i * $skipsize));
        
        // Exit the loop if the position is beyond the end of the file
        if ($pos > $meta->npoints-1) break;
        
        // read from the file
        fseek($fh,$pos*4);
        $val = unpack("f",fread($fh,4));
        
        // calculate the datapoint time
        $time = $meta->start_time + $pos * $meta->interval;

        // add to the data array if its not a nan value
        if (!is_nan($val[1])) $data[] = array($time*1000,$val[1]);
        
        $i++;
     }
