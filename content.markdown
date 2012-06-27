### Introduction

This document summarizes the steps involved in setting up the fonebridge2 T1/E1 PRI-to-Ethernet Bridge with FreeSWITCH via the FreeTDM/libpri/DAHDI stack.

For this lab environment we used: 

####Linux Distro
Debian Squeeze 6.0 
2.6.32-5-amd64

####FreesWITCH
FreeSWITCH Version 1.2.0-rc2 
commit 7dc9a9cacc5366fb04748a89aa7556da8d68e57c
Date:   Mon Jun 25 15:31:58 2012 -0500

####DAHDI/Libpri
DAHDI 2.4.1
Libpri 1.4.12


FreeTDM is a library that implements a high level API for handling I/O and telephony signaling for multiple boards and the fonebridge2 T1/E1 bridge terminations are no exception. 

####I/O Modules

The fonebridge2 uses the DAHDI interface and the DAHDI/IO module in FreeTDM (ftmod_zt) to read and write raw data from/to the board and the FreeTDM application, this module also achieves executing commands in the board.

####Signaling Modules

As explained before the IO module does not interpret any telephony signaling going through and therefore a signaling module is required for this. This document uses the libpri module (ftmod_libpri) for the E1 PRI signaling but FreeTDM supports other PRI stacks as well as MFC-R2 for R2 signaling.

### Installation

FreeTDM is a library and as such can be installed alone, but in this document we consider its installation as part of the FreeSWITCH installation since it was born as a part of FreeSWITCH, and the source code is still part of the FreeSWITCH main repository.



```
$ cd /usr/local/src
$ git clone git://git.freeswitch.org/freeswitch.git
$ cd /usr/local/src/freeswitch
```
If Git was used to download, the configuration files must be built before the first compile. Once this is performed it's not normally required to be performed again:

```
$ ./bootstrap.sh
```

This creates many files including modules.conf, go ahead and uncomment the freetdm line in modules.conf

```
../../libs/freetdm/mod_freetdm
```

```
$ ./configure
$ make
$ make install
```

If you already have installed libpri the configure script will set things up so that ftmod_libpri is built, if you already have built your FreeSWITCH and are only building FreeTDM do as follows:

```
$ cd libs/freetdm
$ ./configure --with-libpri --prefix=/usr/local/freeswitch
$ make
$ make install
```

### Configuration

####The Library

First we must configure the library (the freetdm library) which is done via a simple text file (not XML) located at _/usr/local/freeswitch/freetdm.conf_, when FreeTDM is installed as part of FreeSWITCH. This file configures the IO module and therefore it does not deal with any kind of telephony signaling parameters and we only configure which spans and channels to use, the trunk type, groups, audio gains. This may reflect the configurations you have on your /etc/dahdi/system.conf.

An example for a 4 E1 fonebridge2 board follows:

```
;ISDN PRI with DAHDI                                                                                                           
[span zt FreeTDM1]
name => firstcircuit
number => 1
trunk_type => E1
b-channel=1-15
d-channel=16
b-channel=17-31

[span zt FreeTDM2]
name => secondcircuit
number => 2
trunk_type => E1
b-channel=32-46
d-channel=47
b-channel=48-62

[span zt FreeTDM3]
name => thirdcircuit
number => 3
trunk_type => E1
b-channel=63-77
d-channel=78
b-channel=79-93

[span zt FreeTDM4]
name => fourthcircuit
number => 4
trunk_type => E1
b-channel=94-108
d-channel=109
b-channel=110-124
```

####The signaling

After configuring the library we need to actually tell FreeSWITCH to use it and tell it which signaling settings to use and where to send the incoming calls to.

This is done via a typical FreeSWITCH XML configuration file located autoload_configs/freetdm.conf.xml

Which for our 4 E1 board will look like this:

```
<configuration name="freetdm.conf" description="FreeTDM Configuration">

        <settings>
                <param name="debug" value="0"/>
                <!--<param name="hold-music" value="$${moh_uri}"/>-->
                <!-- Analog global options (they apply to all spans)                                                           
                     Remember you can only choose between either call-swap                                                     
                     or 3-way, not both!                                                                                       
                -->
                <!--<param name="enable-analog-option" value="call-swap"/>-->
                <!--<param name="enable-analog-option" value="3-way"/>-->
                <!--                                                                                                           
                Refuse to load the module if there is configuration errors                                                     
                Defaults to 'no'                                                                                               
                -->
                <!--<param name="fail-on-error" value="no"/>-->
        </settings>

        <!-- Spans will be configured by the libpri module ftmod_libpri -->
        <libpri_spans>
                <span name="FreeTDM1">
                  <param name="node" value="cpe"/>
                  <param name="switch"   value="euroisdn"/>
                  <!-- <param name="opts" value="omit_redirecting_number"/> -->
                  <!-- <param name="dp" value="unknown"/> -->
                  <param name="debug" value="none"/>
                  <!-- <param name="debug"    value="q931_all"/> -->
                  <param name="dialplan" value="XML"/>
                  <param name="context" value="public"/>
                  <param name="l1" value="alaw"/>
                </span>

                <span name="FreeTDM2">
                  <param name="node" value="cpe"/>
                  <param name="switch"   value="euroisdn"/>
                  <!-- <param name="opts" value="omit_redirecting_number"/> -->
                  <!-- <param name="dp" value="unknown"/> -->
                  <param name="debug" value="none"/>
                  <!-- <param name="debug"    value="q931_all"/> -->
                  <param name="dialplan" value="XML"/>
                  <param name="context" value="public"/>
                  <param name="l1" value="alaw"/>
 </span>

                <span name="FreeTDM3">
                  <param name="node" value="cpe"/>
                  <param name="switch"   value="euroisdn"/>
                  <!-- <param name="opts" value="omit_redirecting_number"/> -->
                  <!-- <param name="dp" value="unknown"/> -->
                  <param name="debug" value="none"/>
                  <!-- <param name="debug"    value="q931_all"/> -->
                  <param name="dialplan" value="XML"/>
                  <param name="context" value="public"/>
                  <param name="l1" value="alaw"/>
                </span>

                <span name="FreeTDM4">
                  <param name="node" value="cpe"/>
                  <param name="switch"   value="euroisdn"/>
                  <!-- <param name="opts" value="omit_redirecting_number"/> -->
                  <!-- <param name="dp" value="unknown"/> -->
                  <param name="debug" value="none"/>
                  <!-- <param name="debug"    value="q931_all"/> -->
                  <param name="dialplan" value="XML"/>
                  <param name="context" value="public"/>
                  <param name="l1" value="alaw"/>
                </span>
        </libpri_spans>
</configuration>
```

With this configuration Q931 call setup and digit collection will be handled via ftmod_libpri and you should see calls arrive within the public dialplan context.

Assuming DID 1100 from your provider you could receive a test call as follows:

conf/dialplan/public.xml
```
<extension name="public_pri_extension">
      <condition field="destination_number" expression="^(1100)$">
        <action application="transfer" data="$1 XML default"/>
      </condition>
    </extension>
```

conf/dialplan/default.xml
```
<extension name="pri_test">
      <condition field="destination_number" expression="^1100$">
        <action application="answer"/>
        <action application="sleep" data="1000"/>
        <action application="playback" data="voicemail/vm-goodbye.wav"/>
        <action application="sleep" data="20000"/>
        <action application="hangup"/>
      </condition>
    </extension>
```

### Support or Contact
Check out the documentation at http://support.red-fone.com/ or contact support@red-fone.com and we’ll help you sort it out.