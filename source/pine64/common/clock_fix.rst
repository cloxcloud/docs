.. warning::
    If the session closes unexpectedly, the time and date must be wrong. The problem is that, in order to cut costs, the Real Time Clock (RTC) was left out from this SBC, so every time you reboot date and time will be lost. You can add an RTC, use an NTP server or simply update manually the date. Containers can't modify the system's clock for security issues, so just update the time and date on box0 and the OpenNebula's container will automatically update it. Log in to **box0** and, as root, check and update the time if necessary:
.. prompt:: bash # auto

    # date
    # date -s "2 OCT 2006 18:00:00"
