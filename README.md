# Akka-Actors
 The Actor Model provides a higher level of abstraction for writing concurrent and distributed systems.
 It alleviates the developer from having to deal with explicit locking and thread management, making it easier to write correct concurrent and parallel systems. 

 Use of Actors allows us to:
 Enforce encapsulation without resorting to locks.
 use the model of cooperative entities reacting to signals, changing state and sending signals to each other to drive the whole application forward.

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

 ### 1. Creating an Actor

 -Actors are defined in a hierarchy system. All the actors that share a common configuration will be defined by an ActorSystem.

 - Define an ActorSystem
      ActorSystem system = ActorSystem.create("test-system");
 -The system contains 3 actors:
    *The route guardian actor- having the address “/” which as the name states represent the root of the actor system hierarchy.
    *The user guardian actor- having the address “/user”. This will be the parent of all the actor we define.
    *The system guardian actor- having the adress "/system". This will be the parent of all the actors defined internally by the Akka system.

#### 2. Any Actor will extend the AbstractActor abstract class and implement the createRecieve method for handling the incoming messages from other actors
          public class MyActor extends AbstractActor {
        public Receive createReceive() {
            return receiveBuilder().build();
        }
      }

   This basic actor can receive messages from other actors and will discard them because no matching message patterns are defined in the ReceiveBuilder.
      Now that we’ve created our first actor we should include it in the ActorSystem:
        
          ActorRef readingActorRef 
    = system.actorOf(Props.create(MyActor.class), "my-actor");


  ## 3. Actor Configuration

   -The props class contains the actor configuration.This class is immutable, thus thread-safe, so it can be shared when creating new actors.
   let’s define an actor the will do some text processing. The actor will receive a String object on which it’ll do the processing:
          public class ReadingActor extends AbstractActor {
    private String text;

          public static Props props(String text) {
              return Props.create(ReadingActor.class, text);
          }
          // ...
        }

   Now, to create an instance of this type of actor we just use the props() factory method to pass the String argument to the constructor:

        ActorRef readingActorRef = system.actorOf(
        ReadingActor.props(TEXT), "readingActor");


   ## 4. Actor Messaging
   To interact with each other, the actors can send and receive messages from each other in the system. This message can be any
   type of objects that is immutable.

   It’s a best practice to define the messages inside the actor class. This helps to write code that is easy to understand and know what messages an actor can handle.

   ### 5. Sending Messages
   Inside the Akka actor system messages are sent using methods:
   tell()
   ask()
   forward()

   When you want to send a message and dont expect a response you use the tell() method.

       readingActorRef.tell(new ReadingActor.ReadLines(), ActorRef.noSender());

   The first parameter reprsents the message we want to send to the actor adress readingActorRef.
   The second parameter specifies who the sender is. This is useful when the actor receiving the message needs to send
   a response to an actor other than the sender (for example the parent of the sending actor).

   Usually, we can set the second parameter to null or ActorRef.noSender(), because we don’t expect a reply.
   When we need a response back from an actor, we can use the ask() method:

        CompletableFuture<Object> future = ask(wordCounterActorRef, 
        new WordCounterActor.CountWords(line), 1000).toCompletableFuture();

   When asking an actor for a response and completionStage object is returned so the processing remains non-blocking.
   To return a future object that will contain an exception,we must send a status.failure message to the sender actor.

   This is not done automatically when an actor throws an exception while processing a message and the ask() call will timeout and no 
   reference to the exception will be seen in the logs:

             @Override
             public Receive createReceive() {
                 return receiveBuilder()
                   .match(CountWords.class, r -> {
                       try {
                           int numberOfWords = countWordsFromLine(r.line);
                           getSender().tell(numberOfWords, getSelf());
                       } catch (Exception ex) {
                           getSender().tell(
                            new akka.actor.Status.Failure(ex), getSelf());
                            throw ex;
                       }
                 }).build();
             }


There is also forward() method which is similar to tell() only that the original sender of the message is kept. So the actor forwarding the message only acts as an intermediary actor:

                     printerActorRef.forward(
                     new PrinterActor.PrintFinalResult(totalNumberOfWords), getContext());
       
          
## Receiving Messages 

Each Actor will implement the createReceive() method which handles all incoming messages. The ReceivebuildeR() acts like a switch statement, 
trying to match the received message to the type of message defined.

                     public Receive createReceive() {
                         return receiveBuilder().matchEquals("printit", p -> {
                             System.out.println("The address of this actor is: " + getSelf());
                         }).build();
                     }
When received, a message is put into a FIFO queue, so the messages are handled sequentially.

When we finish using an actor, we kill it by using the stop()method from the ActorRefFactory interface

                   system.stop(myActorRef);

We can use this method to terminate any child actor or the actor itself. It’s important to note stopping is done asynchronously and 
that the current message processing will finish before the actor is terminated. No more incoming messages will be accepted in the actor mailbox.

By stopping a parent actor, we’ll also send a kill signal to all of the child actors that were spawned by it.

When we don’t need the actor system anymore, we can terminate it to free up all the resources and prevent any memory leaks:

                  Future<Terminated> terminateResponse = system.terminate();
 We could also send a PoisonPill message to any actor that we want to kill:

                  myActorRef.tell(PoisonPill.getInstance(), ActorRef.noSender());     

The PoisonPill message will be received by the actor like any other message and put into the queue. The actor will process all the messages until
it gets to the PoisonPill one. Only then the actor will begin the termination process.
                 
   
Another special message used for killing an actor is the Kill message. Unlike the PoisonPill, the actor will throw an ActorKilledException when processing this message:

                myActorRef.tell(Kill.getInstance(), ActorRef.noSender()); 
     
 Best practices when working with Akka:
   1.Use tell() instead of ask() when performance is a concern
   2.when using ask() we should always handle exceptions by sending a Failure message
   3.actors should not share any mutable state
   4.an actor shouldn’t be declared within another actor
   5.actors aren’t stopped automatically when they are no longer referenced. We must explicitly destroy an actor when we don’t need it anymore to prevent memory leaks
   messages used by actors should always be immutable

 

 


  
 
