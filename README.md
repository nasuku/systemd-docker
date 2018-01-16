
This is based on github.com/ibuildthecloud/systemd-docker
Please go to http://github.com/ibuildthecloud/systemd-docker for more info about it.

However - There is one important change in this.
This no longer moves the cgroups (or) notifies the systemd of the PID of the container process.
Instead I run the service as 'simple' systemd service.
The systemd-docker will, in a loop, keep watching for the health of the container.
If there is any error, it exits.

I also specify the  ExecStartPre and ExecStop, ExecStopPost - to take care of any case where the container didnt stop/remove clearly.

Here is an example:

```ini
# more /usr/lib/systemd/system/my.service
[Unit]
Description=my test
After=docker.service
Requires=docker.service

[Service]
ExecStartPre=-/usr/bin/docker kill busybox
ExecStartPre=-/usr/bin/docker rm -f busybox
ExecStart=/usr/pensando/bin/systemd-docker run --name busybox --rm busybox /bin/sh -c "while true; do echo Hello World; sleep 1; done"
ExecStop=/usr/bin/docker stop busybox
ExecStopPost=/usr/bin/docker rm -f busybox
Restart=always
RestartSec=10s
Type=simple
NotifyAccess=all
TimeoutStartSec=120
TimeoutStopSec=15
```

Why different from systemd-docker
---------------------------------
While my production setup uses docker and runs the services with systemd, I wanted ability to unit-test my code.
I started off with running the whole setup with docker (hence using docker in docker). However the cgroup treatment
is really painful. A service running in docker in docker has very odd path in /proc/../cgroup which actually does not exist in the same path if mounting the file.
Hence moving the container to systemd cgroup does not work.

Given such a case, my above solution seems to work for me.
If you see any issues, please open an issue or send a pull-request.

License
-------
[Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)
