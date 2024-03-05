# Akka-Actors
 The Actor Model provides a higher level of abstraction for writing concurrent and distributed systems.
 It alleviates the developer from having to deal with explicit locking and thread management, making it easier to write correct concurrent and parallel systems. 

 ## The Actor Model
 An Actor represents an independent computational unit.
 ### Characteristics of Actor Model
 
  -An Actor encapsulates its state and part of the application logic.<br>
  -Actors interact only through aysnchronous messages.<br>
  -each actor has a unique adress and mailbox in sequential order.<br>
  -The actor system is organized in a tree-like hierarchy.<br>
  -An actor can create other actors , can send messages to other actors and stop itseld or any actor it has created.<br>

 ### Advantages of Actor Model
 
  -The sender thread won’t block to wait for a return value when it sends a message to another actor. <br>
  -You dont have to worry about sycnhronization in a multithreaded environment, Since all messages are processed sequentially.<br>
  -Error Handling-By organizing the actors in a hierarchy, each actor can notify its parent of the failure, 
  so that it can act accordingly.The parent actor can decide to stop or restart the child actors.<br>

 ### Creating an Actor

 -Actors are defined in a hierarchy system. All the actors that share a common configuration will be defined by an ActorSystem.

 - Define an ActorSystem
      ActorSystem system = ActorSystem.create("test-system");
 -The system contains 3 actors:
    *The route guardian actor- having the address “/” which as the name states represent the root of the actor system hierarchy.
    *The user guardian actor- having the address “/user”. This will be the parent of all the actor we define.
    *The system guardian actor- having the adress "/system". This will be the parent of all the actors defined internally by the Akka system.

#### Any Actor will extend the AbstractActor abstract class and implement the createRecieve method for handling the incoming messages from other actors
          public class MyActor extends AbstractActor {
        public Receive createReceive() {
            return receiveBuilder().build();
        }
      }

   This basic actor can receive messages from other actors and will discard them because no matching message patterns are defined in the ReceiveBuilder.
      Now that we’ve created our first actor we should include it in the ActorSystem:
        
          ActorRef readingActorRef 
    = system.actorOf(Props.create(MyActor.class), "my-actor");


  ## Actor Configuration

   -The props class contains the actor configuration.This class is immutable, thus thread-safe, so it can be shared when creating new actors.


 

 


  
 
