# nxlog (http://nxlog-ce.sourceforge.net/)
Docker image based on Ubuntu 14.04 with nxlog version 2.9.1504.

# Usage
Example conf file:

```xml
########################################
# Global directives                    #
########################################
User nxlog
Group nxlog

LogFile /var/log/nxlog/nxlog.log
LogLevel INFO

<Extension syslog>
    Module	xm_syslog
</Extension>

########################################
# Modules                              #
########################################
<Input in>
    Module	im_file

    # We monitor all files matching the wildcard.
    # Every line is read into the $raw_event field.
    File	"/var/log/*.log"

    # Set the $EventTime field usually found in the logs by extracting it with a regexp.
    # If this is not set, the current system time will be used which might be a little off.
    Exec	if $raw_event =~ /(\d\d\d\d\-\d\d-\d\d \d\d:\d\d:\d\d)/ $EventTime = parsedate($1);

    # Now set the severity to something custom. This defaults to 'INFO' if unset.
    Exec	if $raw_event =~ /ERROR/ $Severity = 'ERROR'; \
                else $Severity = 'INFO';

    # The facility can be also set, otherwise the default value is 'USER'.
    Exec	$SyslogFacility = 'AUDIT';

    # The SourceName field is called the TAG in RFC3164 terminology and is usually the process name.
    Exec	$SourceName = 'my_application';

    # It is also possible to rewrite the Hostname if you don't want to use the system's hostname.
    Exec	$Hostname = 'myhost';

    # The Message field is used if present, otherwise the current $raw_event is prepended with the
    # syslog headers.
    # You can do some modifications on the Message if required. Here we add the full path of the
    # source file to the end of message line.
    Exec	$Message = $raw_event + ' [' + file_name() + ']';

    # Now create our RFC3164 compliant syslog line using the fields set above and/or use sensible
    # defaults where possible. The result will be in $raw_event.
    Exec	to_syslog_bsd();
</Input>

<Output out>
    # This module just sends the contents of the $raw_event field to the destination defined here,
    # one UDP packet per message.
    Module	om_udp
    Host	192.168.1.42
    Port	1514
</Output>

########################################
# Routes                               #
########################################
<Route 66>
    Path	in => out
</Route>
```

Create a volume referencing your conf to internal conf file.

```
docker run -it -v $(pwd)/nxlog.conf:/etc/nxlog/nxlog.conf leslau/nxlog
```
