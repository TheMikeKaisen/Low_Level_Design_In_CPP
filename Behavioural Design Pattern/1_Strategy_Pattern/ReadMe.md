# Low Level Design

- Strategy Design pattern
    
    *The Strategy pattern is a behavioral design pattern that defines a family of algorithms, encapsulates each one, and makes them interchangeable. It lets the algorithm vary independently from the clients that use it by delegating the behavior to a separate object instead of hardcoding it into the class.”*
    
    Then add:
    
    - It promotes **composition over inheritance**.
    - It provides **runtime flexibility**, since strategies can be swapped dynamically.
    - It keeps the code **open for extension but closed for modification** (OCP).
    
    Use the Strategy pattern when you have multiple variations of a behavior (like sorting algorithms, payment methods, or robot abilities) and you want to avoid bloated inheritance trees and duplicated code.
    
    ## Example
    
    ![image.png](Low%20Level%20Design%2024ed329c5ef980e09b3fe7fc2b437fbb/image.png)
    
    Robot is the Base Class with 3 functions. It have children → Companion Robot, Worker Robot, Sparrow Robot ( which needs additional function → fly( ) ) and Crow Robot (also fly).
    
    1. Here is a base class → Robot with the functions → walk(), talk() and projection(). Every other robot will have the functionality of walk and talk, but projection is a pure virtual function, which makes Robot an Abstract class. this means that every other child robot will have their own projection which they will override from the base Robot function.
    2. This calls for a problem, there are multiple child robots which can fly.  fly() function will have the same code for every other robot that can fly(lets say). But Base class Robot doesn’t have FLY( ) function. So if there are 10 robots that can fly, then fly( ) function would have to written 10 times. Which is redundant and thus it breaks Don’t repeat yourself ( DRY ) and Open-Closed Principle (OCP). 
    
    ### Using Inheritance to make it a little better
    
    ![image.png](Low%20Level%20Design%2024ed329c5ef980e09b3fe7fc2b437fbb/image%201.png)
    
    Here we have defined another class Flyable Robot with the function fly( ). Now any child class Robot that can fly can inherit fly( ) function from this Flyable Class.
    
    ### Introducing Problem 1
    
    Lets say, we have another classes Jet Robot 1 and Jet Robot 2, which uses Jet Propellers to fly instead of wings.
    
    In that case, the inheritance UML would look more like:
    
    ![image.png](Low%20Level%20Design%2024ed329c5ef980e09b3fe7fc2b437fbb/image%202.png)
    
    ### Introducing Problem 2
    
    Lets say,  these child robot classes can be divided into yet another division based on if they can Talk or Not. Then Inheritance would also include → Talkable Class and Non-Talkable Class.
    
    ![image.png](Low%20Level%20Design%2024ed329c5ef980e09b3fe7fc2b437fbb/image%203.png)
    
    Annnnddddd………
    
    THIS NEVER ENDS!
    
    This is one of the reasons why **Inheritance is not Scalable.**
    
    So what can be a better solution?
    
    ### Strategy Design Pattern → For the Rescue!
    
    Strategy Design Pattern defines a family of algorithm, put them into separate classes so that they are changed at runtime.
    
    ![image.png](Low%20Level%20Design%2024ed329c5ef980e09b3fe7fc2b437fbb/image%204.png)
    
    Yupp! Confusing…..ik ik!
    
    1. So what is done here is called composition ( taking a class as a data member by another class).
    2. ABSTRACT classes have been created and they have their pure virtual functions. So that there is a clear differentiation between Robots that can Talk, Walk and Fly.
    3. Each of these class ( Talkable, Walkable, Flyable ) is takes as a data member ( NOT INHERITED ) by base class “Robot”.
    4. So now, any child Class ( Companion Robot, Sparrow Robot, Crow Robot, Jet Robot ) whatever robot they may be, only need to override one single function, i.e. projection( ) funciton. 
    5. So Karthik what happens to other functions? like fly, talk, etc?
        
        So the thing is, that is exactly why we have created the classes → Walkable, Talkable, and Flyable!. They also have child class → NormalTalk, NoTalk, etc.
        
    6. Now in the main() function, when i am creating the robot object, i can specifically tell which kind of Robot it is: if it can talk or walk or fly.
    
    ```cpp
    Robot* robo = new CompanionRobot(
    													new NormalTalk(), 
    													new NormalWalk(),
    													new NONFlyable()
    													);
    ```
    
    1. Wait a second….but I am taking Walkable, Talkable and Flyable as composition classes inside base class “Robot”, but i am passing NormalTalk, NormalWalk, and NONFlyable function as constructor arguments. How will it work?
        
        That’s where the ultimate power of POLYMORPHISM comes in. See, Abstract Classes have a propery called runtime polymorphism. An Abstract Class Pointer can point to any of its derived class at runtime.
        
        That’s why Talkable class is going to point to NormalTalk() function here, and Walkable Class is going to point Normal Walk function, and so on.