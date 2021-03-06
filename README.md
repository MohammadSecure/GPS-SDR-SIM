# GPS-SDR-SIM

GPS-SDR-SIM generates GPS baseband signal data streams, which can be converted 
to RF using software-defined radio (SDR) platforms, such as 
[bladeRF](http://nuand.com/), [HackRF](https://github.com/mossmann/hackrf/wiki), and [USRP](http://www.ettus.com/).

### Windows build instructions

1. Start Visual Studio.
2. Create an empty project for a console application.
3. On the Solution Explorer at right, add "gpssim.c" and "getopt.c" to the Souce Files folder.
4. Select "Release" in Solution Configurations drop-down list.
5. Build the solution.

### Building with GCC

```
$ gcc gpssim.c -lm -O3 -o gps-sdr-sim
```

### Generating the GPS signal file

A user-defined trajectory can be specified in either a CSV file, which contains 
the Earth-centered Earth-fixed (ECEF) user positions, or an NMEA GGA stream.
The sampling rate of the user motion has to be 10Hz.
The user is also able to assign a static location directly through the command line.

The user specifies the GPS satellite constellation through a GPS broadcast 
ephemeris file. The daily GPS broadcast ephemers file (brdc) is a merge of the
individual site navigation files into one. The archive for the daily file is:

[ftp://cddis.gsfc.nasa.gov/gnss/data/daily/](ftp://cddis.gsfc.nasa.gov/gnss/data/daily/)

These files are then used to generate the simulated pseudorange and
Doppler for the GPS satellites in view. This simulated range data is 
then used to generate the digitized I/Q samples for the GPS signal.

The bladeRF command line interface requires I/Q pairs stored as signed 
16-bit integers, while the hackrf_transfer and gps-sdr-sim-uhd.py
support signed bytes.

HackRF and bladeRF require 2.6 MHz sample rate, while the USRP2 requires
2.5 MHz (an even integral decimator of 100 MHz).

The simulation start time can be specified if the corresponding set of ephemerides
is available. Otherwise the first time of ephemeris in the RINEX navigation file
is selected.

The maximum simulation duration time is defined by USER_MOTION_SIZE to 
prevent the output file from getting too large.

The output file size can be reduced by using "-b 1" option to store 
four 1-bit I/Q samples into a single byte. 
You can use [bladeplayer](https://github.com/osqzss/gps-sdr-sim/tree/master/player)
for bladeRF to playback the compressed file.

```
Usage: gps-sdr-sim [options]
Options:
  -e <gps_nav>     RINEX navigation file for GPS ephemerides (required)
  -u <user_motion> User motion file (dynamic mode)
  -g <nmea_gga>    NMEA GGA stream (dynamic mode)
  -l <location>    Lat,Lon,Hgt (static mode) e.g. 30.286502,120.032669,100
  -t <date,time>   Scenario start time YYYY/MM/DD,hh:mm:ss
  -T <date,time>   Overwrite TOC and TOE to scenario start time
  -d <duration>    Duration [sec] (dynamic mode max: 300 static mode max: 86400)
  -o <output>      I/Q sampling data file (default: gpssim.bin)
  -s <frequency>   Sampling frequency [Hz] (default: 2600000)
  -b <iq_bits>     I/Q data format [1/8/16] (default: 16)
  -i               Disable ionospheric delay for spacecraft scenario
  -v               Show details about simulated channels
```

The user motion can be specified in either dynamic or static mode:

```
> gps-sdr-sim -e brdc3540.14n -u circle.csv
```

```
> gps-sdr-sim -e brdc3540.14n -g triumphv3.txt
```

```
> gps-sdr-sim -e brdc3540.14n -l 30.286502,120.032669,100
```
Note: In use of Hackrf One SHould:

```
> gps-sdr-sim -e brdc3540.14n -u circle.csv -b 8 
```

```
> gps-sdr-sim -e brdc3540.14n -g triumphv3.txt -b 8
```

```
> gps-sdr-sim -e brdc3540.14n -l 30.286502,120.032669,100 -b 8
```

Note: For Create Virtual Dynamic roud in GPS Reciver you should:
```
1. Go to maps.google.com, and use the MyMaps feature to create a named map.
2. Add as many Placemarks as needed, including pertinent information for each.
3. Export route data by Export to KML file
4. Convert kml file to csv and use: gps-sdr-sim -e brdc3540.14n -u circle.csv
```

### Transmitting the samples

The TX port of a particular SDR platform is connected to the GPS receiver 
under test through a DC block and a fixed 50-60dB attenuator.

The simulated GPS signal file, named "gpssim.bin", can be loaded
into the bladeRF for playback as shown below:

```
set frequency 1575.42M
set samplerate 2.6M
set bandwidth 2.5M
set txvga1 -25
cal lms
cal dc tx
tx config file=gpssim.bin format=bin
tx start
```

You can also execute these commands via the `bladeRF-cli` script option as below:

```
> bladeRF-cli -s bladerf.script
```

For the HackRF:

```
> hackrf_transfer -t gpssim.bin -f 1575420000 -s 2600000 -a 1 -x 0
```

For UHD supported devices (tested with USRP2 only):

```
> gps-sdr-sim-uhd.py -t gpssim.bin -s 2500000 -x 0
```


### License

Copyright &copy; 2015 Takuji Ebinuma  
Distributed under the [MIT License](http://www.opensource.org/licenses/mit-license.php).


#### Note -> https://github.com/osqzss/gps-sdr-sim/issues/26
Hello again
I thought I should let you know this:
Via playing with frequencies, I thought maybe my HackRF has some frequency drift/offset.
Here's a picture of sending on 1,575,420,000 HZ :
http://i.imgur.com/rFUYAbM.png

And here's when sending on 1,575,414,000 HZ:
http://i.imgur.com/JMkG8Yt.png
(This is the closest thing I've managed to get yet, although it still does not lock)

I think my problem lies in software, because the HackRF is not faulty (I have tried sending on a lot of frequencies and it works)

Mehdi

For the HackRF:

```
> hackrf_transfer -t gpssim.bin -f 1575414000 -s 2600000 -a 1 -x 47 -R
