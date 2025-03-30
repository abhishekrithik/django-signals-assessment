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
```
