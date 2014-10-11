# Smart Outlets

## Description

This document describes the protocol used for communication between the SmartOutlets network (`coordinator`) and the SMRTControl device (`client`).

## Packet Format

Each packet consists of three components: 

 * a device type ID
 * a packet ID
 * a message ID
 * a data segment

The devie type ID identifies the message as a SmartOutlets packet.  This should always be `0x03` for all communication between the `coordinator` and the `client`.

The packet ID contains is one byte that uniquely identifies the packet type.

The message ID is an arbitrary identifier for the message.  In all requests, it is chosen by the `client` and in all responses it is the ID of the message being responded to.  For example, a response to a request with message ID 0x1F would also have a message ID of 0x1F.  This is so the `client` can associate the response it receives with the specific request it issued.

The data segment contains zero or more bytes that are specified by the packet's type.

| Start bit | End bit | Field                                            |
|:---------:|:-------:| ------------------------------------------------ |
| 0         | 7       | Device type ID                                   |
| 8         | 15      | Packet ID                                        |
| 16        | 23      | Message ID                                       |
| 24        | __...__ | Data segment                                     |


## Commands/Requests

This section describes the commands and requests sent to the `coordinator` to perform an action or request information.


### REQ_OUTLETS

__ID__: 0x00

Request a list of outlets and their power states.  Can be used to fetch information about specific outlets or all outlets in the network.

The data segment contains an [outlet list](#outlet-list).  The power state bit in each entry is ignored.  To request all outlets, send an empty list (ie. `0x00`).

__Response__: [RESP_OUTLETS](#resp_outlets)


### REQ_POWER_STATS

__ID__: 0x01

Request power consumption statistics for outlets.  Can be used to fetch stats for a specific group of outlets or all outlets in the network.

The data segment contains an start timestamp, an end timestamp, and an [outlet list](#outlet-list).  the start and end timestamps do not need to be specified;  setting either to `0x00` will default to `forever_ago` and `now`, respectively.  The power state bit in each outlet list entry is ignored.  To request power stats for all outlets, send an empty list (ie. `0x00`).

| Start bit | End bit | Field                                            |
|:---------:|:-------:| ------------------------------------------------ |
| 0         | 31      | Start timestamp                                  |
| 32        | 63      | End timestamp                                    |
| 64        | ...     | Outlet list                                      |

__Response__: [RESP_POWER_STATS](#resp_power_stats)


### REQ_SCHEDULE

__ID__: 0x02

Request automation schedule.  Fetches the entire automation schedule.

No data segment.

__Response__: [RESP_SCHEDULE](#resp_schedule)


### SET_POWER_STATE

__ID__: 0x03

Set the power state of one or more outlets.

The data segment contains an [outlet list](#outlet-list).  Sets the power state of the outlet according to the power state bit.

__Response__: [ACK](#ack)


### SCHEDULE_TASKS

__ID__: 0x04

Add one or more tasks to the automation schedule.

Data segment contains an [automation task list](#automation-list).  The task ID fields are ignored (this is set the `coordinator`).

__Response__: [ACK](#ack)


### UNSCHEDULE_TASKS

__ID__: 0x05

Remove a scheduled automation task.

Data segment contains a [task ID list](#task-id-list) of tasks to unschedule.

__Response__: [ACK](#ack)


## Responses

This section describes the responses from the `coordinator`.

### ACK

__ID__: 0x80

A message send by either the `coordinator` or the `exterior` in acknowledgement of various messages.

__Requests__: [SET_POWER_STATE](#set_power_state), [SCHEDULE_TASKS](#schedule_tasks), [UNSCHEDULE_TASKS](#unschedule_tasks)


### RESP_OUTLETS

__ID__: 0x81

Respond with list of outlets and their power states.

Data segment contains an [outlet list](#outlet-list).

__Request__: [REQ_OUTLETS](#req_outlets)


### RESP_POWER_STATS

__ID__: 0x82

Respond with list of outlets and their power stats.

Data segment is WIP.

__Request__: [REQ_POWER_STATS](#req_power_stats)


### RESP_SCHEDULE

__ID__: 0x83

Response containing scheduled automation tasks.

Data segment contains an [automation task list](#automation-list).

__Request__: [REQ_SCHEDULE](#req_schedule)


## Lists

A list is a sequence of entries preceded by the number of entries in the list.  Below is a table illustrating the form of the list

| Bytes     | 0      | 1 - _n_    |  (_n_+1) - 2_n_ | ... |
| --------- | ------ | ---------- | --------------- | --- | 
| __Field__ | Length | First item | Second item     | ... |

### Outlet list

An outlet list is a sequence of entries that describe an outlet and its power state.  Each entry is composed of a 7 bit identifier and 1 bit indicating the power state of the outlet.

| Start bit | End bit | Field                                            |
|:---------:|:-------:| ------------------------------------------------ |
| 0         | 6       | Outlet ID                                        |
| 7         | 7       | Outlet power state.  `0` for OFF and `1` for ON. |


### Automation list

An automation list is a sequence of entries that describe an automation task.  Each entry is composed of an 8 bit task ID, a 7 bit outlet ID, 1 bit describing the power state, a 4 byte unix timestamp, and an options field.  Currently the options field is unspecified but will likely be used to options like how often a task repeats.

| Start bit | End bit | Field                                            |
|:---------:|:-------:| ------------------------------------------------ |
| 0         | 7       | Task ID                                          |
| 8         | 14      | Outlet ID                                        |
| 15        | 15      | Outlet power state.  `0` for OFF and `1` for ON. |
| 16        | 47      | Unix timestamp for first occurrence.             |
| 48        | 55      | Options                                          |


### Task list

A task list is similar to an [automation list](#automation-list) but contains only a list of task IDs.  This is primarily used for un-scheduling specific tasks.

| Start bit | End bit | Field                                            |
|:---------:|:-------:| ------------------------------------------------ |
| 0         | 7       | Task ID                                          |

### Power stats list

I'm not sure how I want to implement sending power statistics yet.
