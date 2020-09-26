# Task Score
Task Score provides the score for an activity to the Behavior Designer (BD) Utility Selector based on the weights you assign to  attributes. The BD Utility Selector then runs the activity with the highest score.   For example, a Retrieve HealthPack task could be configured with a high weighting for low health and a high weighting for a HealthPack being nearby.  As the health rating gets worse, the Retrieve HealthPack score gets higher  and eventually becomes the highest rated task for the Utility Selector.  Using weighted attribute task scores can provide more natural and intelligent behavior for an agent's decisions rather than having a tree of binary decisions. 

# Overview of Components

These components work with the Opsive Behavior Utility Selector:

- *Task Score* - This BD task provides the score for an activity to the BD Utility Selector based on the weights you assign to various attributes. The Utility Selector then runs the activity with the highest score.  Any float Behavior Designer global variable can be used as a component of the score for a task.
- *Anger* - This Unity component increases the anger attribute of the Agent in the BD Attribute Manager when the agent is attacked. The Attribute Manager can be configured to decrease anger over time.  The initial value for anger in the Attribute Manager can range from zero for a passive agent to 100 for an aggresive agent.  
- *Distance* - This Unity component determines if an object is visible, calculates the distance, and updates a Behavior Designer global variable. Rather than having a cut-off based on in or out of field of view, this determines visibility based on a combination of angle and distance.  The further to the side the object is, the lower the distance it is visible, while an object directly in front of the agent is visible further away.  
- *Behavior Variables* - This Unity component updates variables in Behavior Designer such as Ammo, Weapon, Health, and Anger.

# 1. Task Score Component

Task Score is a Behavior Designer task which returns the score for a particular task group.  The Utility Selector will then run the task with the highest score.  

### Parameters
Any float Behavior designer global variable can be a component of the task score.  All attributes should be scaled from 0 to 100.0f.  You can set a weighting from 0 to 1.0f for each attribute.  The  score is the sum of each attribute’s weight times the attribute’s value.    To provide a consistent basis for scoring between activities, the sum of the weights should be equal to 1.0f for a high priority task, 0.9f for medium priority, and 0.8f for a low priority activity.

### Reversed Scale
The Utility Selector chooses the task with the highest values, so sometimes you need to reverse the scale for  a parameter to have it increase utility.  For example, as a player’s health gets lower, you might want to give a higher result for getting a healthpack.  You can do this by assigning a negative weight.  This is handled as a special case that reverses the scale (100-value) and applies the absolute value of the weight. 

### Sample Parameters
*Target Close* -  100=close, 1=far, 0=not visible. Value is 100 - distance (capped at 99).  The target can be healthpack, weapon, ammo, player, etc.  
*Explore* -  Increases if no valuable targets have been found.  100=None found, 99=Useful targets are far, 0=Useful targets are near.  Value = (avg(distance) + min(distance))/2  The higher this value, the more useful exploring is.  
*Target Health* - 100=healthy, 0=dead  
*Health* - 100=healthy, 0=dead  
*Anger* - 100=angry, 0=calm.  Damage to the agent increases anger.  Time decreases anger (based on the rate set in Attribute Manager)  
*Weapons* - 0 none, 10 club, 20 knife, 30 sword, 40 pistol, 50 shotgun, 60 automatic   
*Ammo* - 100=full, 0=empty  

### Setup
1. In the Behavior Designer edit window, add a Task Score Task for each activity under Utility Selector and connect it to the Utility Selector with the actions connected below it.  
1. In the BD Variables tab, add the Global Variables you want for your project (potentially the items in Sample Parameters above).  
1. In the BD Inspection window for each Task Score task, add the Global Variables you want for this task and add the weight for each variable. 
1. Setup Behavior Variables as described in that section.
1. If you want to use Anger in your score, setup Anger as described in that section.

### Example

In this example, we are equal distance from the player and from the healthpack and our health is low.  We have two tasks to score:  1) Seek Healthpack, and 2) Attack Player.  The attribute values are:  
*Health=20, Target Close(HealthPack)=70, Target Close(Player)=70*  

*Task Score for “Seek Heathpack”*  
Weight Set to:  Health= -0.6, Target Close(HealthPack)=0.4  
Score= (80 * 0.6) + (70 * 0.4) = 76  (Note the Health score is reversed to 80 because the weight is negative.  The lower our health, the higher we want the score.)  
  
*Task Score for “Attack Player”*  
Weight set to:  Health= 0.5, Target Close(Player)=0.4  
Score= (20 * 0.5) + (70 * 0.4) = 38  (Note that the sum of weights is 0.9, so if everything is equal (equal distance to targets and health=50), the Seek Healthpack will be selected since it has total weights of 1.0.)  

# 2. Anger Component

Anger is a component that increases the anger attribute when the agent receives damage and updates the Anger attribute in Attribute Manager.

### Setup
Anger - Add the Anger component to your Agent.  Add an Anger attribute to your agent’s Attribute Manager with min/max of 0,100 and set auto decrement (if you choose).  Set the initial value to zero for a passive agent and to 100 for an aggresive agent or any value in between.  

# 3. Distance Component  
The distance component has a list of objects and their types (healthpack, player, ammo, etc).  It will determine if an object is visible by the agent and if so will calculate the distance.  An object type, such as HealhPack, can have multiple objects, and the distance value will be for the closest visible object of that type.  Any distance over 99 is capped at 99.  100 is used to indicate not visible.

### Field of Vision
Rather than having a fixed cutoff for whether an item is in or out of field of vision, there is a table with distance viewable based on angle.  Objects toward the center can be seen further off, and off to the side must be closer to be considered visible.

# 4. Behavior Variables Component

### Setup  
Add the Behavior Variables component to your agent.


