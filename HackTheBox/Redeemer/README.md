# Redeemer — HTB Machine Writeup

![Platform](https://img.shields.io/badge/Platform-Hack%20The%20Box-9FEF00?logo=hackthebox&logoColor=black)
![Category](https://img.shields.io/badge/Category-Redis%20Enumeration-red)
![OS](https://img.shields.io/badge/OS-Linux-blue?logo=linux)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## Machine Information

| Field | Details |
|---|---|
| Machine | Redeemer |
| Platform | Hack The Box |
| Type | Starting Point |
| Category | Redis Enumeration |
| OS | Linux |
| Difficulty | Very Easy |
| Service | Redis |
| Port | 6379/tcp |
| Redis Version | 5.0.7 |
| Status | Pwned |

## Achievement

[View Hack The Box Achievement](https://labs.hackthebox.com/achievement/machine/4058304/472)

## 1. Introduction

The Redeemer machine focused on identifying and accessing an exposed Redis database service.

The investigation involved network reconnaissance, service identification, Redis enumeration, and database key analysis to retrieve the root flag.

The primary objectives were:

- Identify open ports and running services.
- Discover the Redis database service.
- Connect using the Redis command-line interface.
- Enumerate server information and databases.
- Identify stored keys and retrieve the root flag.

## 2. Network Reconnaissance

I started by scanning the target machine to identify open ports and running services.

```bash
sudo nmap -sV -sC -p- TARGET_IP
```

The scan identified the following service:

| Port | State | Service | Version |
|---|---|---|---|
| 6379/tcp | Open | Redis | 5.0.7 |

Port 6379 is the default port used by Redis.

The discovery of this service established Redis as the primary attack surface for further investigation.

## 3. Understanding the Redis Service

Redis is an in-memory NoSQL database that stores information in key-value pairs.

It is commonly used for:

- Application caching.
- Session management.
- Real-time data processing.
- Database-backed application services.

The service was accessible on the target machine, so I proceeded with Redis enumeration.

## 4. Connecting to Redis

I used the Redis command-line utility to establish a connection with the target service.

```bash
redis-cli -h TARGET_IP -p 6379
```

The connection was successful, providing access to the Redis command interface.

The `-h` option specifies the target hostname or IP address, while `-p` specifies the port.

## 5. Enumerating Redis Server Information

After establishing the connection, I executed the `INFO` command to retrieve server information.

```redis
INFO
```

The output provided information about the Redis server, including its version, configuration, and database statistics.

**Identified Redis Version:**

```text
5.0.7
```

This confirmed that the target was running Redis version 5.0.7.

## 6. Database Enumeration

Redis supports multiple logical databases.

I selected database 0 using:

```redis
SELECT 0
```

Next, I checked the number of keys stored in the selected database:

```redis
DBSIZE
```

**Result:**

```text
4
```

The database contained four keys.

To identify the stored keys, I executed:

```redis
KEYS *
```

This returned the available keys in database 0.

The enumeration helped identify the information stored within the Redis instance.

## 7. Retrieving the Root Flag

After identifying the relevant key, I used the Redis `GET` command to retrieve its stored value.

```redis
GET flag
```

The command returned the root flag, confirming successful completion of the machine.

## 8. Security Findings

The primary security concern identified during this assessment was the accessibility of the Redis service without adequate authentication.

An improperly secured Redis deployment can expose sensitive application information and database contents.

**Potential Security Impacts:**

- Unauthorized database access.
- Exposure of sensitive information.
- Compromise of application session data.
- Potential manipulation of stored application data.
- Increased risk of further compromise.

## 9. Security Recommendations

To improve Redis security, the following measures are recommended:

1. Restrict Redis access to trusted network interfaces.
2. Enable authentication and appropriate Redis access controls.
3. Configure firewall rules to restrict access to port 6379.
4. Avoid exposing Redis directly to the public internet.
5. Use TLS for protected network communication where supported.
6. Monitor Redis access logs and suspicious activity.
7. Maintain supported and updated Redis versions.

## 10. Key Takeaways

This machine reinforced several important cybersecurity concepts:

1. Network reconnaissance using Nmap.
2. Identifying exposed database services.
3. Understanding Redis architecture and key-value storage.
4. Connecting to Redis using `redis-cli`.
5. Retrieving server information using `INFO`.
6. Enumerating logical databases and stored keys.
7. Understanding the security implications of unauthenticated database access.

## 11. Conclusion

The Redeemer machine provided practical experience in Redis service enumeration and database security assessment.

By identifying the exposed Redis service, establishing a connection, examining server information, and enumerating database keys, I successfully retrieved the root flag.

The challenge demonstrated how insufficiently protected database services can expose sensitive information and highlighted the importance of proper authentication, network restrictions, and access controls.

**Machine completed successfully — HTB Starting Point: Redeemer.**
