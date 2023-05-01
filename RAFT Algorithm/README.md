# CS3700 Project 6: Distributed Key-Value Database

This project is a distributed, replicated key-value datastore that implements Raft consensus protocol to maintain
consistency between replicas. The database supports two API calls of `put(key, value)` and `get(key)` which enables
the client to store a key and retrieve it, respectively. By utilizing Raft, we can ensure that our process will run
multiple times in parallel and that the datastore will function across replica failures.

## Approach

Beginning with the starter code, the suggested implementation approach was followed, where basic support for put and
get requests were added, with responses being fail messages. The next step was to implement the Raft election protocol,
which was initiated when the current time exceeded the last heartbeat plus the timeout value. If there was a current 
leader at that point, there a heartbeat message would be sent out in the form of an append entry request, which 
would reset the timer. On the other hand, if there was no leader, the current replica would
transition to a candidate state, increment its term and send out "vote request" messages to the other replicas.
Once the messages were received, they would vote for the candidate and continue the process until a replica won by
majority and be declared the new leader. For the keys and values, each time a put request was received by a leader, it
would store the pair in the state machine, which allowed it retrieve the value if a get request was received. To send
data to replicas, we implemented a quorum system where all a majority of replicas would have to agree to commit an
update, which we did by sending various commit accept and reject messages. We also added functionality to retrying
failed commits in unreliable networks by resending them, and sending multiples logs at once through batching.

## Challenges

The most challenging part of the process was debugging because there were many print messages and moving parts within
the program that it was difficult to pin down where an exact issue was and how to resolve it. An instance of this
occurring was when votes were not being recorded and a leader not being elected, which resulted in hours of debugging,
and realizing that it had come from the incorrect handling of the vote response messages. Regarding all the 
functionality we implemented, the most difficult part was the quorum because we had to determine when an update could
be committed and how the messages sent between the replicas could reflect this. We resolved this issue by testing 
various approaches of storing logs and sending different commit messages until our desired result was achieved.

## Properties/Features

The program supports message types of get, put, vote_request, vote_response and append_entry and more for the various 
functionalities. It also has good to exceptional performance for many of the tests, especially the simple and partition
ones where minimal messages are sent between replicas and response time is short.

## Testing

The main testing methodology used was running the test config files, which could be analyzed to determine what each test
did. Within the code, the logging library was utilized to print distinguished statements of various replica fields