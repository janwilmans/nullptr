
# This is a test for a new blog post

 - bla
 - bla
 
 [nullptr.nl](nullptr.nl)
 
```c++
  #include <memory>
  #include <mutex>

  template <typename T>
  class mt_shared_ptr
  {
    using ptr_t = std::shared_ptr<T>;
  public:
    mt_shared_ptr() = default;
    mt_shared_ptr(const mt_shared_ptr& t) : ptr(t.share()) {}
    mt_shared_ptr(mt_shared_ptr&& other) : ptr(other.ptr)
    {
      other.ptr.reset();
    }

    mt_shared_ptr& operator=(const mt_shared_ptr& other)
    {
      if (this != &other)
      {
        ptr_t p = ptr;
        ptr_t q = other.share();
        std::lock_guard<std::mutex> lock(mutex);
        ptr = q;
      }
      return *this;
    }

    // no locking needed since moving implies the rhs is not accessed
    mt_shared_ptr& operator=(mt_shared_ptr&& other)
    {
      if (this != &other)
      {
        ptr = std::move(other.ptr);
      }
      return *this;
    }

    // we make sure we do not (possibly) run the destructor of the object that ptr references while holding the lock
    mt_shared_ptr & operator=(ptr_t p) // shared_ptr assignment
    {
      std::lock_guard<std::mutex> lock(mutex);
      ptr.swap(p);
      return *this;
    }

    mt_shared_ptr(const ptr_t& p) : ptr(p) {} // constructor taking a shared_ptr<T>

    /*
     * enable this only as a temporary measure to migrate code in steps
     * calling -> directly promotes an easy-to-use-incorrectly-pattern, because ptr can be re-assigned between operator-> calls
    ptr_t operator->() const
    {
      std::lock_guard<std::mutex> lock(mutex);
      if (!ptr) throw std::runtime_error("mt_shared_ptr<> nullptr dereference");
      return ptr;
    }
    */

    bool is_lock_free() const noexcept { return false;  }

    ptr_t share() const
    {
      std::lock_guard<std::mutex> lock(mutex);
      return ptr;
    }

    void reset()
    {
      ptr_t delayRelease;
      std::lock_guard<std::mutex> lock(mutex);
      delayRelease = ptr;
      ptr.reset();
    }

    void reset(ptr_t p)
    {
      std::lock_guard<std::mutex> lock(mutex);
      ptr.swap(p);
    }

  private:
    mutable std::mutex mutex;
    ptr_t ptr;
  };
```
