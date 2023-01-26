---
layout: posts
title:  "Implementation of the Stable Marriage Algorithm."
date:   2018-04-12 20:46:34 +0100
categories: edtech, coding, programming, python, computing, computer-science
share: true
---

Ever been faced with the challenge of pairing people based on their individual preferences? I can think of numerous situations in which such a task might arise, for example, in pairing up co-workers for a task or pairing up students for an activity.

If each student were to (1) list out their peers they would 'propose' to work with (in order of preference), and (2) list out their peers in order of who they would 'accept' proposals from (also in order of preference) the Stable Marriage Algorithm would allow us to find a solution such that it would not be preferable for any two students to swap partners - in other words that the matching found by the algorithm is said to be 'stable'. 

Consider that four students list out who they would like to work with as follows. Let's call this the Proposers Table.

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg .tg-le8v{background-color:#c0c0c0;vertical-align:top}
.tg .tg-yw4l{vertical-align:top}
</style>
<table class="tg">
  <tr>
    <th class="tg-le8v">James</th>
    <th class="tg-yw4l">James</th>
    <th class="tg-yw4l">Floriane</th>
    <th class="tg-yw4l">Jemery</th>
    <th class="tg-yw4l">Matthew</th>
  </tr>
  <tr>
    <td class="tg-le8v">Floriane</td>
    <td class="tg-yw4l">Jemery</td>
    <td class="tg-yw4l">Matthew</td>
    <td class="tg-yw4l">James</td>
    <td class="tg-yw4l">Floriane</td>
  </tr>
  <tr>
    <td class="tg-le8v">Jeremy</td>
    <td class="tg-yw4l">Matthew</td>
    <td class="tg-yw4l">Floriane</td>
    <td class="tg-yw4l">Jemery</td>
    <td class="tg-yw4l">Jemery</td>
  </tr>
  <tr>
    <td class="tg-le8v">Matthew</td>
    <td class="tg-yw4l">Jemery</td>
    <td class="tg-yw4l">Floriane</td>
    <td class="tg-yw4l">James</td>
    <td class="tg-yw4l">Matthew</td>
  </tr>
</table>

Consider also that the students list out who they would be willing to accept proposals from, in order of preference. Let's call this the Acceptors Table.

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg .tg-us36{vertical-align:top}
.tg .tg-xxzo{background-color:#c0c0c0;vertical-align:top}
</style>
<table class="tg">
  <tr>
    <th class="tg-xxzo">James</th>
    <th class="tg-us36">Floriane</th>
    <th class="tg-us36">James</th>
    <th class="tg-us36">Jemery</th>
    <th class="tg-us36">Matthew</th>
  </tr>
  <tr>
    <td class="tg-xxzo">Floriane</td>
    <td class="tg-us36">Matthew</td>
    <td class="tg-us36">James</td>
    <td class="tg-us36">Floriane</td>
    <td class="tg-us36">Jemery</td>
  </tr>
  <tr>
    <td class="tg-xxzo">Jeremy</td>
    <td class="tg-us36">James</td>
    <td class="tg-us36">Jemery</td>
    <td class="tg-us36">Floriane</td>
    <td class="tg-us36">Matthew</td>
  </tr>
  <tr>
    <td class="tg-xxzo">Matthew</td>
    <td class="tg-us36">Floriane</td>
    <td class="tg-us36">Jemery</td>
    <td class="tg-us36">James</td>
    <td class="tg-us36">Matthew</td>
  </tr>
</table>

For simplicity, we will allow the students to propose to themselves (i.e. to work with themselves). The proposers and accepters table may be encoded as follows:

{% highlight python %}
Acceptors Table: [[1, 2, 3, 4], [3, 4, 1, 2], [4, 2, 3, 1], [3, 2, 1, 4]]
Proposers Table: [[2, 1, 3, 4], [4, 1, 2, 3], [1, 3, 2, 4], [2, 3, 1, 4]]
{% endhighlight %}

Now, diving into the code. I have implemented the algorithm using three classes. First the Pool Class, which will hold and maintain the list of engagements.

{% highlight python %}
# Pool Class :: holds engagements
class Pool:
    def __init__(self, acceptors):
        """
        Construct an array which will hold the engagements. Instatiate each maximum preference number that 
        """
        self.engagements = np.empty(shape=len(acceptors))
        self.engagements.fill(np.nan) 

    def new_engagement(self,acceptor,proposer):
        """
        Update (replace) the engagement in the pool 
        """
        if proposer in self.engagements:
            print(proposer, "in position", self.engagements.tolist().index(proposer)+1, "set to NaN")
            self.engagements[self.engagements.tolist().index(proposer)] = np.nan

        self.engagements[acceptor-1] = proposer
 
    def is_complete(self):
        """
        Return True if complete
        """
        if (np.isnan(self.engagements).any()):
            return False
        else:
            return True

    def get_current_engagement(self,acceptor): 
        """
        Return the current engagement for a acceptor
        """
        return self.engagements[acceptor-1]

    def get_all_engagements(self):
        """
        Return all the current engagements
        """        
        return self.engagements

{% endhighlight %}

The Acceptor Class will hold the preferences of the person being proposed to, return prefereces and decide if a proposal is accepted or rejected, depending on their current engagement status. The is_proposal_accepted method will return True if either the member being proposed to has no current engagement or if the proposing member is higher up in their list of preferences than their current engagement, otherwise it will return False.

{% highlight python %}
# Acceptor Class :: holds the acceptor preferences
class Acceptor:
    def __init__(self,values):
        """
        Construct the acceptor preferences
        """
        self.values = values

    def get_preference_number(self,acceptor,proposer):
        """
        Return the preference of the acceptor for the proposer passed
        """
        #print(self.values[acceptor-1])
        if proposer in self.values[acceptor-1]:
            return self.values[acceptor-1].index(proposer)+1
        else: 
            return 0

    def is_proposal_accepted(self,acceptor,proposer):
        """
        If proposer is in accepter preferences return true else return false
        """
        if debug: (print("acceptor preference of proposal", self.get_preference_number(acceptor,proposer)))
        if debug: (print("acceptor currently engaged to", pool_object.get_current_engagement(acceptor)))
        if debug: (print("acceptor preference of current engagement", self.get_preference_number(acceptor,pool_object.get_current_engagement(acceptor))))

        if (np.isnan(pool_object.get_current_engagement(acceptor)) and (self.get_preference_number(acceptor,proposer)!=0)):
            return True
        
        if (self.get_preference_number(acceptor,proposer) < self.get_preference_number(acceptor,pool_object.get_current_engagement(acceptor))): 
            return True 
        else:
            return False
{% endhighlight %}

The Proposer Class will hold the proposers preferences. The get_proposal method will return the next proposal and will be called until all of the members are engaged.

{% highlight python %}
# Proposer Class :: holds the proposer preferences
class Proposer:
    def __init__(self, values):
        """
        Construct the proposer preferences
        """
        self.values = values

    def get_proposal(self,proposer,iteration):
        """
        Return the acceptor value (proposal to try) for the proposer and iteration passed
        """
        #return self.values.iloc[proposer,iteration]
        return self.values[proposer][iteration]
{% endhighlight %}

Next we call the Pool, Acceptor and Proposer constructor methods to instantiate the class objects. We use the encoded data above.

{% highlight python %}

# Instantiate the Acceptor and Proposer class passing the encoded data to the class constructor
accepter_object = Acceptor(acceptors_table)
proposer_object = Proposer(proposers_table)

print("Acceptors Table:", accepter_object.values)
print("Proposers Table:", proposer_object.values)

# Instantiate the pool class
pool_object = Pool(np.unique(acceptors_table))
if debug: print("Pool Object:", pool_object.get_all_engagements())
{% endhighlight %}

Now the interesting part. We iterate through the proposals in the proposers_table (by row then column) calling the acceptor object each time to determine if a proposal is accepted. If a proposal is accepted then the pool object which maintains the list of engagements is updated. After each iteration, we check if we have a complete one-to-one bipartite mapping (i.e. each member is engaged) and if so we break out and print the final list of engagements. It's that simple.

{% highlight python %}
def stable_marriage():
    for iteration in range(len(proposers_table)):
        print("\n Round:", iteration+1)
        for proposer in range(len(proposers_table[iteration])):
            print("PROPOSAL:", proposer+1, "---->", proposers_table[proposer][iteration])              
            if accepter_object.is_proposal_accepted(proposer_object.get_proposal(proposer,iteration),proposer+1): #if proposal is accepter
                if debug: print("PROPOSAL ACCEPTED")
                pool_object.new_engagement(proposer_object.get_proposal(proposer,iteration),proposer+1)
            else:
                if debug: print("PROPOSAL FAILED")
            print("ENGAGEMENTS:", pool_object.get_all_engagements())

            if pool_object.is_complete():
                return pool_object.get_all_engagements()

print("\n FINAL ENGAGEMENTS:", stable_marriage())
{% endhighlight %}

When we run the algorithm, this is the output.

{% highlight python %}

 Round: 1
PROPOSAL: 1 ----> 2
ENGAGEMENTS: [nan  1. nan nan]
PROPOSAL: 2 ----> 4
ENGAGEMENTS: [nan  1. nan  2.]
PROPOSAL: 3 ----> 1
ENGAGEMENTS: [ 3.  1. nan  2.]
PROPOSAL: 4 ----> 2
ENGAGEMENTS: [ 3.  4. nan  2.]

 Round: 2
PROPOSAL: 1 ----> 1
ENGAGEMENTS: [ 1.  4. nan  2.]
PROPOSAL: 2 ----> 1
ENGAGEMENTS: [ 1.  4. nan  2.]
PROPOSAL: 3 ----> 3
ENGAGEMENTS: [1. 4. 3. 2.]

 FINAL ENGAGEMENTS: [1. 4. 3. 2.]

{% endhighlight %}

So, our matchings are that James works with James, Floriane works with Matthew, Jemery works with Jemery, and Matthew works with Floriane (by symmetry). There are no two students in the group that would prefer to swap partners - hence the solution is said to be 'stable'.

A full version of the code is available to download from my GitHub page <a href="https://github.com/jamesdhope/teaching-lecturing-resources/blob/master/stableGroups.py">here.</a>

For more information on my implementation of the Stable Marriage Algorithm DM @jamesdhope or email <a href="mailto:{{ site.email }}">{{ site.email }}</a>.