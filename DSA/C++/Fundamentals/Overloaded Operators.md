# Overloaded Operators in C++

Operator overloading means we're giving these existing operators new meanings when used with user-defined types like classes.

---

## Overloaded Insertion Operator (`<<`)

Used to define how a custom object (like `Post`) should be printed using output streams (e.g., `std::cout`).

- It allows us to do: `std::cout << post;`
- `cout` global object of type `ostream` and represents a standard output stream (like the terminal/console)
- `sout` is a parameter name used to store and keep track of the output stream passed by reference, so that the output goes in the same stream.
- This operator must be implemented as a non-member (usually friend) function.
- First parameter: reference to an `ostream` (e.g., `std::cout`)
- Second parameter: const reference to the object being printed
- Return type: `ostream&` to allow chaining like `std::cout << obj1 << obj2;`

```cpp
ostream& operator<<(ostream& sout, const Post& post) { //here the sout would be storing the value of current terminal and acts as an alias for the cout as it is passed by reference
    sout  << "Post#: " << post.getPostID()
          << ", likes#: " << post.getNumLikes()
          << ", connect level: " << post.getConnectLevel();
    return sout;
}
```

**Example usage:**

```cpp
Post p(101, 56, 3);
std::cout << p;
```

---

## Overloaded Assignment Operator (`=`)

Used to define how one object of a class should be assigned to another.

- Called when you do: `obj1 = obj2;`
- Required if your class handles dynamic memory or needs deep copying.
- Typically checks for self-assignment and then copies member data.
- Takes a const reference to the source object.
- Returns `*this` to allow assignment chaining (`a = b = c`);

```cpp
class Post {
private:
    int postID;
    int numLikes;
    int connectLevel;
public:
    // Constructor
    Post(int id = 0, int likes = 0, int level = 0)
        : postID(id), numLikes(likes), connectLevel(level) {}

    // Overloaded assignment operator
    Post& operator=(const Post& other) {
        if (this != &other) {  // check for self-assignment
            postID = other.postID;
            numLikes = other.numLikes;
            connectLevel = other.connectLevel;
        }
        return *this;
    }

    // Example getters
    int getPostID() const { return postID; }
    int getNumLikes() const { return numLikes; }
    int getConnectLevel() const { return connectLevel; }
};
```

**Example usage:**

```cpp
Post a(100, 20, 2);
Post b;
b = a; // Now b has the same data as a
```
