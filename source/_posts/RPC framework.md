# Remote Procedure Call (RPC)

### What is Remote Procedure Call (RPC)?

Remote Procedure Call is a software communication protocol that one program can use to request a service from a program located in another computer on a network without having to understand the network's details. RPC is used to call other processes on the remote systems like a local system. A procedure call is also sometimes known as a *function call* or a *subroutine call*.

RPC uses the client-server model. The requesting program is a client, and the service-providing program is the server. Like a local procedure call, an RPC is a synchronous operation requiring the requesting program to be suspended until the results of the remote procedure are returned. However, the use of lightweight processes or threads that share the same address space enables multiple RPCs to be performed concurrently.

The interface definition language (IDL) -- the specification language used to describe a software component's application programming interface (API) -- is commonly used in Remote Procedure Call software. In this case, IDL provides a bridge between the machines at either end of the link that may be using different operating systems (OSes) and computer languages.

### What does RPC do?

When program statements that use the RPC framework are compiled into an executable program, a stub is included in the compiled code that acts as the representative of the remote procedure code. When the program is run and the procedure call is issued, the stub receives the request and forwards it to a client runtime program in the local computer. The first time the client stub is invoked, it contacts a name server to determine the transport address where the server resides.

The client runtime program has the knowledge of how to address the remote computer and server application and sends the message across the network that requests the remote procedure. Similarly, the server includes a runtime program and stub that interface with the remote procedure itself. Response-request protocols are returned the same way.

### How does RPC work?

When a remote procedure call is invoked, the calling environment is suspended, the procedure parameters are transferred across the network to the environment where the procedure is to execute, and the procedure is then executed in that environment.

When the procedure finishes, the results are transferred back to the calling environment, where execution resumes as if returning from a regular procedure call.

During an RPC, the following steps take place:

1. The client calls the client stub. The call is a local procedure call with parameters pushed onto the stack in the normal way.
2. The client stub packs the procedure parameters into a message and makes a system call to send the message. The packing of the procedure parameters is called marshalling.
3. The client's local OS sends the message from the client machine to the remote server machine.
4. The server OS passes the incoming packets to the server stub.
5. The server stub unpacks the parameters -- called *unmarshalling* -- from the message.
6. When the server procedure is finished, it returns to the server stub, which marshals the return values into a message. The server stub then hands the message to the transport layer.
7. The transport layer sends the resulting message back to the client transport layer, which hands the message back to the client stub.
8. The client stub unmarshalls the return parameters, and execution returns to the caller.

### Types of RPC

There are several RPC models and distributed computing implementations. A popular model and implementation is the Open Software Foundation's (OSF) Distributed Computing Environment (DCE). The Institute of Electrical and Electronics Engineers (IEEE) defines RPC in its ISO Remote Procedure Call Specification, ISO/IEC CD 11578 N6561, ISO/IEC, November 1991.

Examples of RPC configurations include the following:

- The normal method of operation where the client makes a call and doesn't continue until the server returns the reply.
- The client makes a call and continues with its own processing. The server doesn't reply.
- A facility for sending several client nonblocking calls in one batch.
- RPC clients have a broadcast facility, i.e., they can send messages to many servers and then receive all the resulting replies.
- The client makes a nonblocking client/server call; the server signals the call is completed by calling a procedure associated with the client.

RPC spans the transport layer and the application layer in the Open Systems Interconnection (OSI) model of network communication. RPC makes it easier to develop an application that includes multiple programs distributed in a network. Alternative methods for client-server communication include message queueing and IBM's Advanced Program-to-Program Communication (APPC).

### Pros and cons of RPC

Though it boasts a wide range of benefits, there are certainly a share of pitfalls that those who use RPC should be aware of.

Here are some of the advantages RPC provides for developers and application managers:

- Helps clients communicate with servers via the traditional use of procedure calls in high-level languages.
- Can be used in a distributed environment, as well as the local environment.
- Supports process-oriented and thread-oriented models.
- Hides the internal message-passing mechanism from the user.
- Requires only minimal effort to rewrite and redevelop the code.
- Provides abstraction, i.e., the message-passing nature of network communication is hidden from the user.
- Omits many of the protocol layers to improve performance.

On the other hand, some of the disadvantages of RPC include the following:

- The client and server use different execution environments for their respective routines, and the use of resources (e.g., files) is also more complex. Consequently, RPC systems aren't always suited for transferring large amounts of data.
- RPC is highly vulnerable to failure because it involves a communication system, another machine and another process.
- There is no uniform standard for RPC; it can be implemented in a variety of ways.
- RPC is only interaction-based, and as such, it doesn't offer any flexibility when it comes to hardware architecture.





[What Is Remote Procedure Call (RPC)? Definition from SearchAppArchitecture (techtarget.com)](https://www.techtarget.com/searchapparchitecture/definition/Remote-Procedure-Call-RPC)

[RPC是什么，看完你就知道了 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/187560185)

