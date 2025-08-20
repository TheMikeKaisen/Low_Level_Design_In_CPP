# Low Level Design

- Observer Pattern
    
    ![observerPatternUML.jpg](Low%20Level%20Design%2024ed329c5ef980e09b3fe7fc2b437fbb/observerPatternUML.jpg)
    
    ![image.png](Low%20Level%20Design%2024ed329c5ef980e09b3fe7fc2b437fbb/image.png)
    
    # The Observer Design Pattern in C++:  YouTube Subscription Model
    
    Imagine you have a favorite YouTube channel. You want to know immediately when they upload a new video, without constantly checking their page. This is exactly the problem the **Observer design pattern** solves in programming. It's a way for one object (like a YouTube channel) to automatically tell many other objects (like its subscribers) when something important happens, without needing to know all the details about each subscriber. This creates a "one-to-many dependency" and keeps different parts of your code neatly separated, or "loosely coupled".
    
    ## The Cast of Characters in Your Code
    
    The Observer pattern involves a few key players:
    
    ### 1. The `ISubscriber` Interface (The "Subscriber" Contract)
    
    C++
    
    ```cpp
    class ISubscriber {
    public:
        virtual void update() = 0;
        virtual ~ISubscriber() {}  // virtual destructor for interface
    };
    ```
    
    - **What it is:** This is an **abstract class**, which acts like a blueprint or a contract. Any class that wants to be a "subscriber" (an observer) must promise to implement the
        
        `update()` method.
        
    - **Its Role:** It defines *how* a subscriber will receive notifications. When the YouTube channel (the "Subject") has something new, it will call this `update()` method on all its subscribers. The `virtual ~ISubscriber() {}` is a virtual destructor, important for ensuring proper cleanup of objects that inherit from this interface.
    
    ### 2. The `IChannel` Interface (The "Channel" Contract)
    
    C++
    
    ```cpp
    class IChannel {
    public:
        virtual void subscribe(ISubscriber* subscriber) = 0;
        virtual void unsubscribe(ISubscriber* subscriber) = 0;
        virtual void notifySubscribers() = 0;
        virtual ~IChannel() {}
    };
    ```
    
    - **What it is:** This is another **abstract class**, defining the contract for anything that can be "observed" (a subject or publisher).
    - **Its Role:** It outlines the basic actions a YouTube channel must provide:
        - `subscribe(ISubscriber* subscriber)`: How a new subscriber signs up.
        - `unsubscribe(ISubscriber* subscriber)`: How a subscriber cancels their subscription.
        - `notifySubscribers()`: How the channel broadcasts updates to everyone who's subscribed.
        - The `virtual ~IChannel() {}` is also a virtual destructor, ensuring proper cleanup.
    
    ### 3. The `Channel` Class (The Concrete YouTube Channel - Our "Subject")
    
    C++
    
    ```cpp
    class Channel : public IChannel {
    private:
        vector<ISubscriber*> subscribers;  // list of subscribers
        string name; // name of the youtube channel
        string latestVideo;               // latest uploaded video title
    public:
        Channel(const string& name) {
            this->name = name;
        }
    
        // Add a subscriber (avoid duplicates)
        void subscribe(ISubscriber* subscriber) override {
            // find pointer wil reach subscribers.end() if there is no given subscriber in the vector
            // thus we can avoid adding duplicate subscribers
            if (find(subscribers.begin(), subscribers.end(), subscriber) == subscribers.end()) {
                subscribers.push_back(subscriber);
            }
        }
    
        // Remove a subscriber if present
        void unsubscribe(ISubscriber* subscriber) override {
            auto it = find(subscribers.begin(), subscribers.end(), subscriber);
            if (it!= subscribers.end()) {
                subscribers.erase(it);
            }
        }
    
        // Notify all subscribers of the latest video when a new video is added
        void notifySubscribers() override {
            for (ISubscriber* sub : subscribers) {
                sub->update();
            }
        }
    
        // Upload a new video and notify all subscribers
        void uploadVideo(const string& title) {
            latestVideo = title;
            cout << "\n[" << name << " uploaded \"" << title << "\"]\n";
            notifySubscribers(); // this is how you can actually upload notify all subscribers
        }
    
        // Read video data
        string getVideoData() {
            return "\nCheckout our new Video : " + latestVideo + "\n";
        }
    };
    ```
    
    - **What it is:** This is the actual YouTube channel, inheriting from `IChannel`. It holds the channel's specific information (like its
        
        `name` and `latestVideo`).
        
    - **Its Role:**
        - `vector<ISubscriber*> subscribers;`: This is the channel's list of all its current viewers (subscribers).
        - `subscribe()`: When someone subscribes, their `ISubscriber` pointer is added to this `subscribers` list. The `if (find(...) == subscribers.end())` part makes sure you don't add the same subscriber twice.
        - `unsubscribe()`: When someone unsubscribes, their pointer is removed from the list.
        - `notifySubscribers()`: This method loops through every `ISubscriber*` in its `subscribers` list and calls their `update()` method. This is how the channel tells everyone about a new video.
        - `uploadVideo(const string& title)`: This is the channel's main action. When a new video is uploaded, the `latestVideo` is updated, and then, crucially, `notifySubscribers()` is called. This means the channel doesn't need to know *who* is subscribed, just that it needs to tell *everyone* on its list.
        - `getVideoData()`: This method allows subscribers to "pull" the new video title from the channel when they are notified.
    
    ### 4. The `Subscriber` Class (The Concrete Viewer - Our "Observer")
    
    ```cpp
    class Subscriber : public ISubscriber {
    private:
        string name;
        Channel* channel;
    public:
        Subscriber(const string& name, Channel* channel) {
            this->name = name;
            this->channel = channel;
        }
    
        // Called by Channel; prints notification message
        void update() override {
            cout << "Hey " << name << "," << this->channel->getVideoData();
        }
    };
    ```
    
    - **What it is:** This is a specific viewer, like "Varun" or "Tarun," inheriting from `ISubscriber`.
    - **Its Role:**
        - `Subscriber(const string& name, Channel* channel)`: When a `Subscriber` object is created, it's given a `name` and a pointer to the `Channel` it wants to watch. This `channel` pointer allows the `Subscriber` to later ask the `Channel` for its `getVideoData()`.
        - `update() override`: This is the method that gets called by the `Channel` when a new video is uploaded. Inside this method, the `Subscriber` uses the `channel` pointer to get the `latestVideo` title and then prints a personalized notification. Each `Subscriber` can react to the update in its own way.
    
    ## Putting It All Together: The `main` Function Walkthrough
    
    The `main` function is where we see the Observer pattern in action:
    
    ```cpp
    int main() {
        // Create a channel and subscribers
        Channel* channel = new Channel("CraftedByKarthik"); // Our YouTube channel is created
    
        Subscriber* subs1 = new Subscriber("Varun", channel); // Varun is created, linked to the channel
        Subscriber* subs2 = new Subscriber("Tarun", channel); // Tarun is created, linked to the channel
    
        // Varun and Tarun subscribe to CoderArmy
        channel->subscribe(subs1); // Varun officially subscribes to the channel
        channel->subscribe(subs2); // Tarun officially subscribes to the channel
    
        // Upload a video: both Varun and Tarun are notified
        channel->uploadVideo("Moonlight Sonata movement 1 Tutorial ");
        // Output:
        //
        // Hey Varun,
        // Checkout our new Video : Moonlight Sonata movement 1 Tutorial
        // Hey Tarun,
        // Checkout our new Video : Moonlight Sonata movement 1 Tutorial
    
        // Varun unsubscribes; Tarun remains subscribed
        channel->unsubscribe(subs1); // Varun cancels his subscription
    
        // Upload another video: only Tarun is notified
        channel->uploadVideo("Decorator Pattern Tutorial");
        // Output:
        //
        // Hey Tarun,
        // Checkout our new Video : Decorator Pattern Tutorial
    
        return 0;
    }
    ```
    
    1. **Channel Creation:** We create a `Channel` object named "CraftedByKarthik". This is our "Subject."
    2. **Subscriber Creation:** We create two `Subscriber` objects, "Varun" and "Tarun." These are our "Observers." Notice that when creating them, we pass the `channel` pointer. This links them to the specific channel they are interested in.
    3. **Subscribing:** `channel->subscribe(subs1);` and `channel->subscribe(subs2);` add Varun and Tarun to the channel's internal list of subscribers. Now, the channel knows *who* to notify, but it doesn't know they are specifically "Varun" or "Tarun," just that they are `ISubscriber`s.
    4. **First Video Upload:** When `channel->uploadVideo("Moonlight Sonata movement 1 Tutorial ");` is called, the channel updates its `latestVideo` and then calls `notifySubscribers()`. This method iterates through its list and calls `update()` on both `subs1` and `subs2`. Both Varun and Tarun receive the notification and print their messages.
    5. **Unsubscribing:** `channel->unsubscribe(subs1);` removes Varun from the channel's subscriber list.
    6. **Second Video Upload:** When `channel->uploadVideo("Decorator Pattern Tutorial");` is called again, the channel updates its video and calls `notifySubscribers()`. This time, only Tarun is in the list, so only `subs2->update()` is called, and only Tarun receives the notification.
    
    This example clearly shows how the `Channel` (Subject) can broadcast updates without being tightly tied to the specific `Subscriber` (Observer) types. It simply relies on the `ISubscriber` interface. This makes the system flexible: you could add new types of subscribers (e.g., a "NotificationApp" subscriber that sends a push notification) without changing the `Channel` class at all!
    
    This pattern is widely used in real-world applications, especially in graphical user interfaces (GUIs) where buttons or menus (Subjects) notify various parts of the application (Observers) when they are clicked.