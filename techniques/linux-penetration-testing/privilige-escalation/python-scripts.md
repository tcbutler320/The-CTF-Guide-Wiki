---
description: A popular language for its easy to use syntax and vast library
---

# Python Scripts

## Add sudoers users to system

```python
#!/usr/bin/env python
import os
import sys
try:
        os.system('echo "username ALL=(ALL:ALL) ALL" >> /etc/sudoers')
except:
        sys.exit()
```

