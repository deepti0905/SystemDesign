# SystemDesign
I am writing down points from https://www.youtube.com/watch?v=bUHFg8CZFws, for my quick reference later on.

Ask Requirement Clarification questions.
It is impossible to solve the problem in 45 min interview. This is mainly to get insight on where interviewer is heading to.
# Things you should focus on
1. Users/Customers -> who and how will they use our system
2. Scale(read and write)-> How our system will handle a growing amount of data
  1. How many read queries per second?
  2. How much data is queried per request??
  3. How many video views are processed per second?
  4. Can there be spikes in traffic?
    1. We must expect response from the interviewer on this or assume some value
3. Performance -> How fast our system must be
4. Cost -> Budget Constraints
