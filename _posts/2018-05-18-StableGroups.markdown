---
layout: posts
title:  "Implementation of the Stable Marriage Algorithm to find 'Stable Groups'"
date:   2018-05-05 20:46:34 +0100
categories: edtech, coding, programming, python, computing, computer-science
share: true
---

In my previous post, I explained my implementation of the Stable Marriage Algorithm to find stable pairs. Taking this implementation one step further, I wanted to use the Stable Marriage Algorithm iteratively, to join stable pairs to other stable pairs to forms groups (or sets). This post explains my implementation of how I have used the Stable Marriage Algorithm to form groups of stable pairs.

<h1>Basic Software Architecture</h1>

We will make use of my implementation of the Stable Marriage Algorithm. As before, we will need class objects for holding the proposer and acceptor preferences, as well as the list of engagements (or pairs). Additionally, we will need a class object to hold the list of pairs for iteration of the algorithm. We will call this the Pools Class and it will hold Pool Class Objects (or the pairs found at each level). 

Another important aspect of this design is that as we will alterate running the Stable Pairs Algorithm over all proposals and then over proposals of proposers that have not been previously matched in a Stable Pair. This will ensure that these unmatched proposers (or orphans) are given a bit more help in being matched in a Stable Pair for each iteration of the algorithm. So the basic architecture will look something like this:

<img src="https://raw.githubusercontent.com/jamesdhope/jamesdhope.github.io/master/_posts/SA2.jpg">

<h1>Example Data</h1>

Let's consider the preferences are now set out as as follows. The preferences will form both the Proposers Table and the Acceptors Table (a mirror copy).

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg .tg-us36{vertical-align:top}
.tg .tg-xxzo{background-color:#c0c0c0;vertical-align:top}
</style>
<table class="tg">
  <tr>
    <th class="tg-xxzo">Johnny</th>
    <th class="tg-us36">Barry</th>
    <th class="tg-us36">Charlie</th>
    <th class="tg-us36">Gilly</th>
    <th class="tg-us36">null</th>
    <th class="tg-us36">null</th>
  </tr>
  <tr>
    <td class="tg-xxzo">Alphie</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
  </tr>
  <tr>
    <td class="tg-xxzo">Barry</td>
    <td class="tg-us36">Alphie</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
  </tr>
  <tr>
    <td class="tg-xxzo">Charlie</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">Alphie</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
  </tr>
  <tr>
    <td class="tg-xxzo">Danny</td>
    <td class="tg-us36">Charlie</td>
    <td class="tg-us36">Elle</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
  </tr>
  <tr>
    <td class="tg-xxzo">Freddie</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">Gilly</td>
    <td class="tg-us36">Danny</td>
  </tr>
  <tr>
    <td class="tg-xxzo">Gilly</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">Johnny</td>
    <td class="tg-us36">Freddie</td>
    <td class="tg-us36">null</td>
  </tr>
  <tr>
    <td class="tg-xxzo">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
    <td class="tg-us36">null</td>
  </tr>
</table>

From this input file, what we would like to be able to produce is a list of sets that is formed from joining stable pairs to other stable pairs. So the sets that would be formed would be as follows:

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg .tg-us36{vertical-align:top}
.tg .tg-xxzo{background-color:#c0c0c0;vertical-align:top}
</style>
<table class="tg">
  <tr>
    <th class="tg-xxzo">Set 1</th>
    <th class="tg-xxzo">Set 2</th>
    <th class="tg-xxzo">Set 3</th>
    <th class="tg-xxzo">Set 4</th>
    <th class="tg-xxzo">Set 5</th>
</tr>
  <tr>
    <th class="tg-us36">Alphie</th>
    <th class="tg-us36">Barry</th>
    <th class="tg-us36">Charlie</th>
    <th class="tg-us36">Danny</th>
    <th class="tg-us36">Gilly</th>

  </tr>
  <tr>
        <th class="tg-us36"></th>
    <th class="tg-us36"></th>
    <th class="tg-us36"></th>
    <th class="tg-us36">Elle</th>
    <th class="tg-us36">Johnny</th>
  </tr>
  <tr>
            <th class="tg-us36"></th>
    <th class="tg-us36"></th>
    <th class="tg-us36"></th>
    <th class="tg-us36"></th>
    <th class="tg-us36">Freddie</th>
  </tr>
</table>

<h1>A Data Structure for holding Stable Pairs</h1>

Now let's dive into the python. First we will need to make use of some libraries.

{% highlight python %}
import sys
import csv
import numpy as np
import pandas as pd
import sets
import copy
{% endhighlight %}

The Pool Class Object will hold all of the Stable Paris that are found for each run of the Stable Pairs Algorithm. The methods for this class object will manage and update the pairings.

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
        #print("entering new engagement")
        if proposer in self.engagements:
            #print("new engagement1")
            self.engagements[self.engagements.tolist().index(proposer)] = np.nan

        if acceptor in self.engagements:
            #print("new engagement2")
            self.engagements[self.engagements.tolist().index(acceptor)] = np.nan

        self.engagements[acceptor-1] = proposer
        self.engagements[proposer-1] = acceptor
        #print("new engagement made")

    def is_complete(self):
        """
        Return True if complete
        """
        if (np.isnan(self.engagements).any()):
            return False
        else:
            return True

    def not_engaged(self,proposer):
        """
        Return True if engaged otherwise False
        """
        if proposer in self.engagements:
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

<h1>A Data Structure for holding Preferences</h1>

The Acceptor Class object will hold all of the preferences. The is_proposal_accepted() is an important method which deserves some attention here. 

For a proposal to be accepted, is must not cause a set size that exceeds the max_set_size. This is handled by the function is_set_size_allowed() which is explained in a bit more detail later. If the proposal would not cause a set size that exceeds the max_set_size then for the proposal to be accepted, one of the two following conditions must be met: 
1. The proposer and acceptor must be either unmatched; they must have listed each other in their preferences; and, they must not have already been paired in a previous iteration of the Stable Marriage Algorithm. Or: 
2. They must have listed each other in their preferences and the proposal must be better than the current proposal. 

{% highlight python %}
# Acceptor Class :: holds the acceptor preferences
class Acceptor:
    def __init__(self,values):
        """
        Construct the acceptor preferences
        """
        self.values = values

    def get_preference_number(self,acceptor,proposer,null_position):
        """
        Return the preference of the acceptor for the proposer passed. Return 0 if value is null or if the preference is not in the list.
        """
        #if (proposer==null_position) or (acceptor==null_position): 
        #    return 0

        if proposer in self.values[acceptor-1]:
            return self.values[acceptor-1].index(proposer)+1
        else: 
            return 0

    def is_proposal_accepted(self,acceptor,proposer,pool_object,pools_object,null_position,orphan_round,max_set_size,names,preference,acceptors_table,proposer_object):
        """
        If proposer is in accepter preferences return true else return false
        """
        if debug: print("position of preference in acceptor table (for proposal):", self.get_preference_number(acceptor,proposer))
        if debug: print("acceptor is currently engaged to:", pool_object.get_current_engagement(acceptor))
        if debug: print("position of preference in acceptor table (for current engagement):", self.get_preference_number(acceptor,pool_object.get_current_engagement(acceptor)))

        #check if the proposal would create a set size that exceeds the maximum set size and if so return false
        if is_set_size_allowed(acceptors_table,pool_object,names,preference,proposer,proposer_object,pools_object,max_set_size)==False:
            return False

        if orphan_round: 
            # If Orphan then If Engagements empty accept, Elseif better than current engagement accept; Else reject (i.e. not listed)
            if (np.isnan(pool_object.get_current_engagement(acceptor)) and np.isnan(pool_object.get_current_engagement(proposer)) and pools_object.is_orphan(proposer) and pools_object.is_valid_engagement(acceptor,proposer) and (self.get_preference_number(acceptor,proposer,null_position)!=0)):
                return True
            elif ((pools_object.is_orphan(proposer) and pools_object.is_valid_engagement(acceptor,proposer) and (self.get_preference_number(acceptor,proposer,null_position)!=0) and (self.get_preference_number(acceptor,proposer,null_position) < self.get_preference_number(acceptor,pool_object.get_current_engagement(acceptor),null_position)))): 
                return True 
            else:
                return False
        else:
            # Same logic as above but do not restrict to orphans
            if (np.isnan(pool_object.get_current_engagement(acceptor)) and np.isnan(pool_object.get_current_engagement(proposer)) and (self.get_preference_number(acceptor,proposer,null_position)!=0) and pools_object.is_valid_engagement(acceptor,proposer)):
                return True
            elif ((pools_object.is_valid_engagement(acceptor,proposer) and (self.get_preference_number(acceptor,proposer,null_position)!=0) and (self.get_preference_number(acceptor,proposer,null_position) < self.get_preference_number(acceptor,pool_object.get_current_engagement(acceptor),null_position)))): 
                return True 
            else:
                return False
{% endhighlight %}

As before, the Proposers Class Object holds the list of proposals. There is no change here.

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
        #print("proposer",proposer,"iteration",iteration,"result",self.values[proposer][iteration])
        return self.values[proposer][iteration]
{% endhighlight %}

<h1>A Data Structure for holding previously found Stable Pairs</h1>

The Pools Class Object will hold the list of Pool Class Objects. The is_valid_engagement() is called to check if a member exists in a Stable Pair in a previous Pool Class Object. 

{% highlight python %}
# Pools Class :: holds pool (engagement) objects  
class Pools:
    def __init__(self):
        """
        Construct the proposer preferences
        """
        self.values = []

    def length(self):
        """
        Return the length of the pools set
        """
        return len(self.values)

    def add_pool(self,pool_object):
        """
        Add pool objects to the Pools object class
        """
        self.values.append(pool_object)
        
    def remove_pool(self):
        """
        Remove last object added to values
        """
        self.values.pop(len(self.values)-1)

    def is_valid_engagement(self,acceptor,proposer):
        """
        If already engaged to proposer in previous iteration; otherwise return True
        """
        for i in range(len(self.values)):   
            if self.values[i].get_current_engagement(acceptor)==proposer: 
                return False #if found don't allow engagement
        
        return True 
        
    def is_orphan(self,proposer):
        """
        Check if the proposer appears on any previous set of engagements (pool) and if so return True to indicate orphan
        """
        for i in range(len(self.values)):   
            if not np.isnan(self.values[i].get_current_engagement(proposer)): 
                return False  #if found don't allow engagement
        return True 

    def get_pool_object(self,pool_object_number):
        """
        Return the pool object from the pools class
        """
        return self.values[pool_object_number]
{% endhighlight %}

<h1>Routines for importing & encoding the preference data</h1>

Next, we have a few functions that import the preferences from file, encode and decode the input data as strings to integer values.

{% highlight python %}

def import_preferences(input_file):
    """
        Read the data from file and return the names and preferences
    """
    with open(input_file) as csvfile:
        preferences = []
        names = []
        reader = csv.reader(csvfile)
        print("\n Input file read from file \n")
        for row in reader:
            data = list(row)
            print("{}".format(data))
            preferences.append(data[1:])    
            names.append(data[0])
        return(preferences[1:], names[1:])

def encode(names,name):
    """
    Return the index of the name in the list of names
    """
    return names.index(name)+1

def return_null_position(names):
    """
    Return tye position of the null value
    """
    return names.index("null")+1

def encode_preferences(preferences,names,no_of_preferences):
    """
    Encode the preferences 
    """
    df = pd.DataFrame(data=preferences)
    
    for i in range(0,no_of_preferences):
        df[i] = df[i].apply(lambda x: encode(names,x))

    return(df.values.tolist())

def dec(names,index):
    """
    Return the index of the name in the list of names
    """
    return names[index-1]

def decode(pairs,names):
    """
    Decode the engagements
    """  
    df = pd.DataFrame(data=pairs)
    df[0] = df[0].fillna(-2)  
    df = pd.DataFrame(data=pairs).astype(int)    
    df[0] = df[0].apply(lambda x: dec(names,x) if x>0 else None) #decode engagements (ignore negative values)
    return(df.values.tolist())

{% endhighlight %}

<h1>Routines for forming sets from stable pairs</h1>

We will some functions to build the Stable Pairs found at each iteration of the Stable Marriage Algorithm into sets. The function build_pairs returns a dataframe with the stable pairs found at each iteration. The function build_groups then loops through all of the pairs and joins these to previously joined pairs. The Python sets library is ideal for this as the union of two sets implicitly removes any duplication.

{% highlight python %}

def build_pairs(names,pools_object):
    """
    Build a list of all stable pairs by fetching pairs from each pool_object in the pools_object
    """
    pairs = pd.DataFrame()
    pairs[0] = names

    for i in range(pools_object.length()):
        pairs[i+1] = np.array(decode(pools_object.get_pool_object(i).get_all_engagements(), names)).flatten() #pd.DataFrame(data=engagements)  

    return(pairs)

def check_if_intersection(set_1,set_2):
    """
    Returns true if an intersection between two lists is found, otherwise false
    """
    set1 = set(set_1)
    set2 = set(set_2)

    if ((set1.intersection(set2) != set()) and (not "null" in set1) and(not "null" in set2)):
        return True
    else:
        return False

def get_union(set_1,set_2):
    """
    Returns the union of two lists
    """
    set1 = set(set_1)
    set2 = set(set_2)
    result = set1.union(set2)
    
    if "null" in result:
        result.remove("null")
    return result

def build_groups(names,pairs,pools_object):
    """
    From the pairs data, build lists of sets and return the set lists (provided there are more than 2 columns)
    """
    sets = []
    pairs = pairs.fillna(value="null")
    
    #only perform traverse if we have a minimum of two columns
    if pairs.shape[1]>1:
        
        #create sets for members at first level
        for i in range(len(names)):
            result = list(get_union(list([pairs[0][i]]),list([pairs[1][i]])))
            if result not in sets:
                sets.append(result)
        #print("output of level1 join", sets)
    
        # Now join level2, level3 etc members and maintain the trees
        for i in range(2,pools_object.length()+1): #or   #pairs.shape[1]
            for j in range(len(names)):
                index_to_join = -1
                index_to_remove = -1
    
                #check if the student is already in a set somewhere else and if so get the index of that set
                for k in range(len(sets)):
                    if pairs[i][j] in sets[k]:
                        index_to_remove = k
                        if debug: print("index to remove", index_to_remove)
    
                #get the index of the set the student is already in 
                for k in range(len(sets)):
                    if pairs[0][j] in sets[k]:
                        index_to_join = k
                        if debug: print("index to join",index_to_join)
    
                if (index_to_join==index_to_remove):
                    #print("idex to join = index to remove")
                    pass
                elif ((index_to_remove!=-1) and (index_to_join!=-1)):
                    sets[index_to_join] = list(get_union(sets[index_to_join],sets[index_to_remove]))
                    sets.pop(index_to_remove) 
    
    df = pd.DataFrame(data=sets)
    df = df.transpose()
    return df 

{% endhighlight %}

<h1>The Stable Pairs Algorithm</h1>

The stable_marriage function will call the stable_pairs function over and over again according to the number of iterations defined. For each iteration, the stable_pairs function is called twice. The first time, we attempt for find stable pairs for each member in the dataset. The second time, we attempt to find stable pairs for members in the dataset that have not yet been paired (at any level).

{% highlight python %}
def stable_pairs(pool_object,pools_object,proposer_object,proposers_table,accepter_object,acceptors_table,no_of_preferences,null_position,orphan_round,max_set_size,names):
    """
    Create stable engagements and return the list
    """
    for preference in range(0,no_of_preferences):
        if debug: print("/n PREFERENCE:", preference+1)
        #print("width",range(len(proposers_table[preference])))
        for proposer in range(len(proposers_table)-1):

            if pool_object.not_engaged(proposer+1):
                if debug: print("PROPOSAL:", proposer+1, "---->", proposers_table[proposer][preference])        
                
                if accepter_object.is_proposal_accepted(proposer_object.get_proposal(proposer,preference),proposer+1,pool_object,pools_object,null_position,orphan_round,max_set_size,names,preference,acceptors_table,proposer_object): #if proposal is accepter
                    if debug: print("PROPOSAL ACCEPTED")
                    pool_object.new_engagement(proposer_object.get_proposal(proposer,preference),proposer+1)
                else:
                    if debug: print("PROPOSAL FAILED")

                #print(pool_object.get_all_engagements())

            if pool_object.is_complete():
                return pool_object

    return pool_object

def stable_marriage(pools_object,proposer_object,proposers_table,accepter_object,acceptors_table,iteration,no_of_preferences,null_position,max_set_size,names):
    """
    Call the stable_pairs function iteratively and update the Pools object
    """
    for i in range(iteration):
        if debug: print("ITERATION:",i+1)
        
        # add pool object with stable pairs
        pool_object = Pool(acceptors_table) #create a pool object
        pools_object.add_pool(stable_pairs(pool_object,pools_object,proposer_object,proposers_table,accepter_object,acceptors_table,no_of_preferences,null_position,False,max_set_size,names)) #call stable pairs
        print("\n Engagements (all members) found at iteration {} \n {}".format(i+1,pool_object.get_all_engagements()))
        
        # add pool object with orphans
        pool_object = Pool(acceptors_table) #create a pool object
        pools_object.add_pool(stable_pairs(pool_object,pools_object,proposer_object,proposers_table,accepter_object,acceptors_table,no_of_preferences,null_position,True,max_set_size,names)) #call stable pairs
        print("\n Engagements (Orpans) found at iteration {} \n {}".format(i+1,pool_object.get_all_engagements()))
{% endhighlight %}

As we find stable_pairs and accept proposals, we must also check that a proposal would not create a set size that exceeds the max_set_size. To do this, we make a deep copy of the Pools and Pool Objects and call the build_groups and build_pairs functions to return a dataframe with the sets. Unfortunately calling this function each time we make a proposal is computationally inefficient but we will need to check.

{% highlight python %}
def is_set_size_allowed(acceptors_table,pool_object,names,preference,proposer,proposer_object,pools_object,max_set_size):
    """
    If engagement creates a set size that exeeds max_set_size return False; otherwise return True 
    """
    # create dummy pool and pools
    pool_test = Pool(acceptors_table)
    pool_test = copy.deepcopy(pool_object)     
    pool_test.new_engagement(proposer_object.get_proposal(proposer,preference),proposer+1)
    
    pools_test = Pools()
    pools_test = copy.deepcopy(pools_object)  
    pools_test.add_pool(pool_test)
    
    if (len(build_groups(names,build_pairs(names,pools_test),pools_test))>max_set_size):        
        #print(build_groups(names,build_pairs(names,pools_object),pools_object))        
        return False
        
    del pool_test
    del pools_test
        
    return True
{% endhighlight %}

Finally we can call the import routines, encode the input data, instantiate the Pools Class Object and call the stable_marriage function to run the algorithm. A few simple tweaks here would make it possible for us to run the program from the Command Line.

{% highlight python %}

def main():

    try:
        input_file = "Preferences4.csv"
        #input_file = sys.argv[1]
        iteration = 2
        #iteration = int(sys.argv[2]) # Number of runs of stable pairs algoirthm. Subsequent runs ignore stable pairs already built.
        no_of_preferences = 5
        #no_of_preferences = int(sys.argv[3]) # Number of preferences to read from input file
        max_set_size = 12
        #max_set_size = int(sys.argv[4]) # Maximum number in a set
        
    except: 
        print("stableGroups.py --[input_file] --[iteration] --[no_of_preferences] --[max_set_size]\n")
        print("--[input_file] \n input file in csv format \n")
        print("--[iteration] \n number of runs of Stable Pairs Algorithm \n")
        print("--[no_of_preferences] \n number of preferences to read from input file \n")
        print("--[max_set_size] \n maximum size of a set \n")
        sys.exit()
    else: pass

    output_file_stable_pairs = 'pools.csv'
    output_file_sets = 'sets.csv'
    
    # Import and Encode the preferences data
    preferences,names = import_preferences(input_file) 
    preferences = encode_preferences(preferences,names,no_of_preferences)
    print("\n Input file with {} preferences read and encoded successfully as \n {}".format(no_of_preferences, preferences))

    null_position = return_null_position(names)
    print("\n Instantiating Acceptor and Proposer objects with input preference data... \n")
    acceptors_table = preferences
    #acceptors_table = [[1,3,2,4],[3,4,1,2],[4,2,3,1],[3,2,1,4]]
    proposers_table = acceptors_table

    # Instantiate the Acceptor and Proposer class objects
    accepter_object = Acceptor(acceptors_table)
    proposer_object = Proposer(proposers_table)
    print("\n Instantiating Pool and Pools objects ready to hold engagements... \n")

    # Instantiate the Pools Class object
    pools_object = Pools()

    # Run the Algorithm
    print("\n Finding Stable Pairs with {} iterations of the algorithm... \n".format(iteration))
    stable_marriage(pools_object,proposer_object,proposers_table,accepter_object,acceptors_table,iteration,no_of_preferences,null_position,max_set_size,names)

    # Write the engagements to file
    pairs = build_pairs(names,pools_object)
    print("\n Writing pairs to file (1st iteration row 1 - 2, 2nd iteration 1 - 3 etc.)\n {}".format(pairs))
    #print("\n Writing pairs to file (1st iteration row 1 - 2, 2nd iteration 1 - 3 etc.)\n")
    pairs.to_csv(output_file_stable_pairs)

    # Convert engagements to sets and write to file
    groups = build_groups(names,pairs,pools_object)
    print("\n Writing sets to file (a set is a column)\n {}".format(groups))
    print("length of groups", len(groups))
    groups.to_csv(output_file_sets)

if __name__ == "__main__":
    main()

{% endhighlight %}

<h1>Running the program</h1>

The output data we get from the input data above is as follows. Note that in the final dataframe the sets are displayed as columns.

{% highlight python %}

 Input file read from file 

['Name', 'Preference_1', 'Preference_2', 'Preference_3', 'Preference_4', 'Preference_5']
['Johnny', 'Barry', 'Charlie', 'Gilly', 'null', 'null']
['Alphie', 'null', 'null', 'null', 'null', 'null']
['Barry', 'Alphie', 'null', 'null', 'null', 'null']
['Charlie', 'null', 'Alphie', 'null', 'null', 'null']
['Danny', 'Charlie', 'Elle', 'null', 'null', 'null']
['Elle', 'Freddie', 'Danny', 'null', 'null', 'null']
['Freddie', 'null', 'null', 'null', 'Gilly', 'Danny']
['Gilly', 'null', 'null', 'Johnny', 'Freddie', 'null']
['null', 'null', 'null', 'null', 'null', 'null']

 Input file with 5 preferences read and encoded successfully as 
 [[3, 4, 8, 9, 9], [9, 9, 9, 9, 9], [2, 9, 9, 9, 9], [9, 2, 9, 9, 9], [4, 6, 9, 9, 9], [7, 5, 9, 9, 9], [9, 9, 9, 8, 5], [9, 9, 1, 7, 9], [9, 9, 9, 9, 9]]

 Instantiating Acceptor and Proposer objects with input preference data... 


 Instantiating Pool and Pools objects ready to hold engagements... 


 Finding Stable Pairs with 2 iterations of the algorithm... 


 Engagements (all members) found at iteration 1 
 [ 8. nan nan nan  6.  5. nan  1. nan]

 Engagements (Orpans) found at iteration 1 
 [nan nan nan nan nan nan  8.  7. nan]

 Engagements (all members) found at iteration 2 
 [nan nan nan nan nan nan nan nan nan]

 Engagements (Orpans) found at iteration 2 
 [nan nan nan nan nan nan nan nan nan]

 Writing pairs to file (1st iteration row 1 - 2, 2nd iteration 1 - 3 etc.)
          0       1        2     3     4
0   Johnny   Gilly     None  None  None
1   Alphie    None     None  None  None
2    Barry    None     None  None  None
3  Charlie    None     None  None  None
4    Danny    Elle     None  None  None
5     Elle   Danny     None  None  None
6  Freddie    None    Gilly  None  None
7    Gilly  Johnny  Freddie  None  None
8     null    None     None  None  None

 Writing sets to file (a set is a column)
         0      1        2      3        4     5
0  Alphie  Barry  Charlie  Danny    Gilly  None
1    None   None     None   Elle   Johnny  None
2    None   None     None   None  Freddie  None
length of groups 3

{% endhighlight %}

So the final output of sets is as follows and we can see that it has observed the very simply list of preferences in our input file:

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:black;}
.tg .tg-us36{vertical-align:top}
.tg .tg-xxzo{background-color:#c0c0c0;vertical-align:top}
</style>
<table class="tg">
  <tr>
    <th class="tg-xxzo">Set 1</th>
    <th class="tg-xxzo">Set 2</th>
    <th class="tg-xxzo">Set 3</th>
    <th class="tg-xxzo">Set 4</th>
    <th class="tg-xxzo">Set 5</th>
</tr>
  <tr>
    <th class="tg-us36">Alphie</th>
    <th class="tg-us36">Barry</th>
    <th class="tg-us36">Charlie</th>
    <th class="tg-us36">Danny</th>
    <th class="tg-us36">Gilly</th>

  </tr>
  <tr>
        <th class="tg-us36"></th>
    <th class="tg-us36"></th>
    <th class="tg-us36"></th>
    <th class="tg-us36">Elle</th>
    <th class="tg-us36">Johnny</th>
  </tr>
  <tr>
            <th class="tg-us36"></th>
    <th class="tg-us36"></th>
    <th class="tg-us36"></th>
    <th class="tg-us36"></th>
    <th class="tg-us36">Freddie</th>
  </tr>
</table>

A full version of the code is available to download from my GitHub page <a href="https://github.com/jamesdhope/teaching-lecturing-resources/blob/master/stableGroupsX4.py">here.</a>

For more information on my implementation of the Stable Marriage Algorithm DM @jamesdhope or email <a href="mailto:{{ site.email }}">{{ site.email }}</a>.
