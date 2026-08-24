Multithreaded STOMP Messaging Server & C++ Client

This project is a full-duplex, multi-user messaging system implementing a lightweight version of the **STOMP (Simple Text Oriented Messaging Protocol)** over TCP sockets. 

It features a high-performance **Java Multithreaded Server** capable of running in either a **Thread-Per-Client (TPC)** or **Non-Blocking Reactor (Java NIO)** model, paired with a concurrent **C++ Client** designed for event reporting, topic subscriptions, and real-time alerts.

I built this project to master socket programming, network protocol design, thread synchronization, and cross-language communication (Java server ↔ C++ client) without relying on high-level networking frameworks.

---

## 💡 System Architecture

```text
 ┌──────────────────────┐                     ┌──────────────────────────────────┐
 │      C++ Client      │   TCP Sockets       │           Java Server            │
 │  (Dual-Thread Model) │ ◄─────────────────► │    (TPC or Reactor Pattern)      │
 │                      │    STOMP Frames     │                                  │
 │ • Socket Reader      │ (CONNECT, SEND,     │ • Reactor (NIO Selectors + Pool) │
 │ • User Console Input │  SUBSCRIBE, etc.)   │ • Thread-Per-Client (TPC)        │
 │ • JSON Event Parser  │                     │ • Pub/Sub Channel Router         │
 └──────────────────────┘                     └──────────────────────────────────┘
1. Java Server (server/)Dual Server Architecture: Supports two distinct concurrency paradigms configurable at runtime:  Thread-Per-Client (BaseServer): Traditional blocking I/O model dedicating a thread per connected client.  Reactor Pattern (Reactor): Non-blocking I/O using Java NIO Selector, dispatching ready read/write events across an ActorThreadPool for scalable concurrent client handling.  STOMP Protocol Engine (bgu.spl.net.api): Encodes and decodes null-byte-terminated STOMP frames (StompMessageEncoderDecoder) and processes protocol state transitions (StompMessagingProtocolImpl).  Pub/Sub Routing & State (ConnectionsImpl, Database): Manages unique connection IDs, topic subscriptions, receipt confirmations, and user authentication with concurrent thread safety.  2. C++ Client (client/)Multi-Threaded Architecture: Uses two concurrent threads—one listening for user console commands and another continuously reading frames from the TCP socket to ensure non-blocking UI interactions.  Event & Frame Management: Parses incoming JSON event files, constructs formatted STOMP messages (CONNECT, SUBSCRIBE, SEND, UNSUBSCRIBE, DISCONNECT), and aggregates summaries of channel updates. Project StructurePlaintext├── server/
│   ├── src/main/java/bgu/spl/net/
│   │   ├── api/             # STOMP protocol & Encoder/Decoder interfaces and logic
│   │   ├── srv/             # Server architectures: Reactor, TPC, and Connections manager
│   │   └── impl/
│   │       ├── stomp/       # StompServer entry point
│   │       └── data/        # User session tracking and thread-safe database state
│   └── pom.xml
└── client/
    ├── include/             # C++ Headers (ConnectionHandler, Event, json.hpp)
    ├── src/                 # StompClient, ConnectionHandler, Event parsing
    ├── data/                # Sample event JSON inputs
    └── makefile
🚀 Getting StartedPrerequisitesJava JDK 11+ & MavenGCC / G++ compiler supporting C++11 or higherMake1. Start the Java ServerNavigate to the server directory and build the project:
Bashcd server
mvn compile
Run the server in either Thread-Per-Client (TPC) or Reactor mode:Bash# Thread-Per-Client mode (Port: 7777, Server type: tpc)
mvn exec:java -Dexec.mainClass="bgu.spl.net.impl.stomp.StompServer" -Dexec.args="7777 tpc"

# Reactor mode (Port: 7777, Server type: reactor)
mvn exec:java -Dexec.mainClass="bgu.spl.net.impl.stomp.StompServer" -Dexec.args="7777 reactor"
2. Compile and Run the C++ ClientIn a separate terminal, navigate to the client directory:Bashcd client
make
Run the compiled binary:Bash./bin/StompWCIClient
💬 Example Client CommandsOnce inside the interactive client shell, you can control your connection and subscriptions:Plaintextlogin 127.0.0.1:7777 user password   # Authenticate with the server
join Germany_Spain                  # Subscribe to a topic/channel
report data/events1.json            # Parse JSON file and broadcast events to the topic
summary Germany_Spain user out.txt  # Export aggregated event reports to a file
logout                              # Gracefully disconnect (sends DISCONNECT with receipt)
🔍 Key Engineering TakeawaysLow-Level Socket Concurrency: Managing frame parsing boundaries (null-byte \0 terminators, headers, body) directly on raw TCP byte streams without third-party frameworks.Reactor vs. TPC Trade-offs: Deep hands-on experience designing both non-blocking I/O event loops (using Java NIO SelectionKey and channels) and thread-per-connection architectures.Cross-Platform Thread Coordination: Synchronizing C++ mutexes and condition variables across socket listener threads alongside concurrent Java server maps (ConcurrentHashMap, atomic primitives).
