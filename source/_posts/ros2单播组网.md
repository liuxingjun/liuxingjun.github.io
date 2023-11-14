---
title: ros2单播组网
date: 2023-11-14 13:39
updated: 2023-11-14 13:39
categories: ros2
---
在`/etc/ros/fastdds.xml`文件内增加如下内容
```xml
<?xml version="1.0" encoding="UTF-8"?>
<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
    <participant profile_name="unicast_connection" is_default_profile="true">
        <rtps>
            <builtin>
                <initialPeersList>
                    <locator>
                        <udpv4>
                            <address>127.0.0.1</address>
                        </udpv4>
                    </locator>
                    <locator>
                        <udpv4>
                            <address>172.19.0.2</address>
                        </udpv4>
                    </locator>
                </initialPeersList>
            </builtin>
            <userTransports>
                <transport_id>lotsOfPeers</transport_id>
            </userTransports>
        </rtps>
    </participant>
</profiles>
```

增加环境变量
```bash
echo 'FASTRTPS_DEFAULT_PROFILES_FILE=/etc/ros/fastdds.xml' > ~/.bashrc
```
