### Distributed Key-Value Database - Raft Consensus
## High Level Approach
For this project, we firstly check if we need to handle any messages that were not responded to because the leader was unknown at the time. Then, we implement the serveral different functions to handle different types of messages, like manage_put (for leader), manage_get (for leader), manage_append_entries (for other replicas (follower)), manage_received_vote (for leader), manage_vote_req (for followers), manage_true_append_msg (for leader) and manage_false_append_msg (for leader) and we place them in an if clause in the main run function. We also have manage_heartbeat function to send periodic heartbeats to all followers in order to maintain their authority and to avoid election timeouts. For handling messages received, we firstly change the state to follower if the term is stale and reset any timeouts. We implement many different send functions like send, send_ok, and send_fail, sending_append_entries, broadcast_req_vote, manage_vote_req to handle response to the message source. Then we also have other main functions that handle the voting process in accordance with the Raft protocol like manage_received_vote, manage_req_vote, and broadcast_req_vote to broadcast a vote to all other replicas and manage a received vote and a requested vote for an election of leader from the client. The leader election in our main loop will begin if election timeout reached or there is no leader.  We first check for an election timeout in the run function, then the candidate with the most votes will be elected. Once a replica transition to a candidate state due to having an election timeout, it will broadcast the vote request to all other replicas. The replicas will respond on the vote with a response message that has voteGranted=True if approved and voteGranted=False if rejected. We follow closely through the Raft Consensus Algorithm Paper (https://raft.github.io/raft.pdf), implementing AppendEntries RPC, RequestVote RPC, rules for server and its state. At first, we implement the put function to just put the key-value pair in a dictionary that represents the key and value database, but later on, we decided to improve our implementation by logging the put, send append entries, and if it's sent back to the leader as accepted then we proceed to commit the put requests stored in the log. Fail messages are sent back if the replica's leader is unknown. Requests can be redirect to leader if it's not sent to the current leader.

## Challenges Faced
Because we divided the project into small parts and each of us work on one of them, so we had trouble time when merging all the parts together and making sure the logics for each part don't break the others (for example not causing one test fail while others pass). We also discover edge cases and have to modify our code to bypass those. Another challenge is that our Raft implementation sending too many messages, which caused some of the tests to fail (advanced tests). We believe that the reason of this is our code was missing a batching implementation that prevents sending replica messages on every client request, but instead wait up to 10ms to see if other client requests come in. 

## Features
For this project, we try to implemented as many helper functions as possible to keep the code clean and organized. For example our implementation of append entries has many support functions like sending append entries to a replica, to a given replica and to all the replicas. We also have different functions to handle different types of messages received, and the helper function reset_timeout to reset the election and leader timeouts since we reuse this code many times in our implementation. We run a manage_heartbeat function throughout the process so if the replica is the leader, it will continuously send heartbeats (empty AppendEntries) when 150 to 200 ms has passed since the last heartbeat was sent.
We follow closely through the Raft Consensus Algorithm Paper (https://raft.github.io/raft.pdf), implementing AppendEntries RPC, RequestVote RPC, rules for server and its state. At first, we implement the put function to just put the key-value pair in a dictionary that represents the key and value database, but later on, we decided to improve our implementation by logging the put, send append entries, and if it's sent back to the leader as accepted then we proceed to commit the put requests stored in the log.

## Testing
For testing, we tried to print out the transaction log, the keys and values, and as well as the current term and state to make sure that our election run smoothly and able to handle the voting process correctly like the Raft protocol. We also made sure our code is able to run the configs test and pass them. IF any of them was fail, then we observed the print out information to track down the problems. We export the output from the test to a text file and we scan through it to debug for most of our issues.
