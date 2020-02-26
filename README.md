# SystemDesign
I am writing down points from https://www.youtube.com/watch?v=bUHFg8CZFws, for my quick reference later on.

Ask Requirement Clarification questions.
It is impossible to solve the problem in 45 min interview. This is mainly to get insight on where interviewer is heading to.
# Things you should focus on
* Users/Customers -> who and how will they use our system
* Scale(read and write)-> How our system will handle a growing amount of data
  * How many read queries per second?
  * How much data is queried per request??
  * How many video views are processed per second?
  * Can there be spikes in traffic?
      * We must expect response from the interviewer on this or assume some value
* Performance -> How fast our system must be
  * What is expected write to read data delay?
    * can we consider this in batch processing or real time?
  * What is expected p99 latency for the read queries?
    * should we precache data before it is read to reduce the latency
* Cost -> Budget Constraints
  * Minimize cost of development -> use open source technologies
  * Minimize cost of maintenance -> use public cloud
  
**Not getting requirements clarified is a big no sign from any interviewer, basically this helps you set the functional and non functional requirements from now on**
* Functional Requirement
  * What system will support?
    * Here is a hint from the video creator to write these as simple functions
    * This would help you in finalizing the right parameters without much thinking on Http Request and Response syntax (Guideline think simple and build up)
* Non Functional Requirement
 **usually interviewer will not tell us what non functional requirement is. They would say it say it should be scalable and as fast as possible. It is difficult to achieve both together and we would need to achieve the tradeoffs**
    * how a system is supposed to be
    * Scalable 10000 video views per second
    * Highly Performant (10 ms time to view total view count of a video)
    * Highly Available (survives hardware/network failues, no single point of failure)
*Time to mention the CAP Theorm. Write the non functional requirements on the board as well. This will help you choose amongst different technologies*

    
# Now time for a simple design on the board
![Basic Diagram](https://github.com/deepti0905/SystemDesign/blob/master/Basic%20diagram.PNG)
   





