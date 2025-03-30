# Django Signals Assessment

## 1. Are Django Signals Synchronous or Asynchronous?
By default, **Django signals are synchronous**. This means that the execution of the signal handler blocks the main program until the handler is finished.

### Proof (Blocking Execution)
The following code introduces a delay in the signal to prove that signals block execution:

```python
import time
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

@receiver(post_save, sender=User)
def slow_signal(sender, instance, **kwargs):
    print("Signal started...")
    time.sleep(5)  # Delays execution by 5 seconds
    print("Signal finished after 5 seconds.")

# Testing
user = User(username="testuser")
print("Before saving user...")
user.save()  # This should block for 5 seconds before proceeding
print("After saving user...")
```


## 2. Do Django Signals Run in the Same Thread?  
Yes, **Django signals execute in the same thread as the caller**.  

### **Proof (Printing the Thread Name)**  
To verify that Django signals execute in the same thread, we can print the **current thread's name** when the signal runs:  

```python
import threading
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

@receiver(post_save, sender=User)
def check_thread(sender, instance, **kwargs):
    print(f"Signal is running in thread: {threading.current_thread().name}")

# Testing
print(f"Main thread: {threading.current_thread().name}")
User.objects.create(username="testuser", email="test@example.com")
```


## 3. Do Django Signals Run in the Same Database Transaction?  
By default, **Django signals do not necessarily run in the same database transaction** as the caller. This means that if an error occurs inside a signal, it **does not automatically roll back the main transaction** unless explicitly handled.

### **Proof (Demonstrating Signal Execution and Transaction Behavior)**  
Let's test this by raising an exception inside a signal and checking whether the database transaction rolls back.

#### **Code:**
```python
from django.db import transaction
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

@receiver(post_save, sender=User)
def signal_with_exception(sender, instance, **kwargs):
    print("Signal started...")
    raise Exception("This is a test error!")  # This will cause an error
    print("Signal finished...")  # This line will never execute

# Testing Transaction Behavior
try:
    with transaction.atomic():  # Ensures atomicity
        User.objects.create(username="rollback_test_user", email="rollback@example.com")
except Exception as e:
    print(f"Exception caught: {e}")

# Checking if user was actually saved
if User.objects.filter(username="rollback_test_user").exists():
    print("User was saved despite the signal error!")
else:
    print("User was NOT saved due to atomic transaction rollback.")
```


# Custom Classes in Python

## Rectangle Class Implementation  

### Requirements:
- An instance of the `Rectangle` class requires `length: int` and `width: int` to be initialized.
- We can iterate over an instance of the `Rectangle` class.
- When iterated, it should return:
  ```json
  {"length": <VALUE_OF_LENGTH>}
  {"width": <VALUE_OF_WIDTH>}
  ```

### Implementation:

```python
class Rectangle:
    def __init__(self, length: int, width: int):
        self.length = length
        self.width = width

    def __iter__(self):
        yield {"length": self.length}
        yield {"width": self.width}

# Testing the iteration
rect = Rectangle(10, 5)
for dimension in rect:
    print(dimension)  # Expected Output: {'length': 10}, {'width': 5}
```

### Explanation:
- The class requires `length` and `width` during initialization.
- The `__iter__` method allows us to iterate over an instance.
- It **yields** first the `length` in the required format, then the `width`.

