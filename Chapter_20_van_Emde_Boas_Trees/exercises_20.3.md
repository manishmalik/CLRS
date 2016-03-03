## 20.3 The van Emde Boas tree

### 20.3-1

> Modify vEB trees to support duplicate keys.

```python
class VanEmdeBoasTree:
    def __init__(self, u):
        self.u = u
        temp = u
        bit_num = -1
        while temp > 0:
            temp >>= 1
            bit_num += 1
        self.sqrt_h = 1 << ((bit_num + 1) // 2)
        self.sqrt_l = 1 << (bit_num // 2)
        self.min = None
        self.max = None
        self.min_cnt = 0
        self.max_cnt = 0
        if not self.is_leaf():
            self.summary = VanEmdeBoasTree(self.sqrt_h)
            self.cluster = []
            for _ in xrange(self.sqrt_h):
                self.cluster.append(VanEmdeBoasTree(self.sqrt_l))

    def is_leaf(self):
        return self.u == 2

    def high(self, x):
        return x / self.sqrt_l

    def low(self, x):
        return x % self.sqrt_l

    def index(self, x, y):
        return x * self.sqrt_l + y

    def minimum(self):
        return self.min

    def maximum(self):
        return self.max

    def member(self, x):
        if x == self.min or x == self.max:
            return True
        if self.is_leaf():
            return False
        return self.member(self.cluster[self.high(x)], self.low(x))

    def successor(self, x):
        if self.is_leaf():
            if x == 0 and self.max == 1:
                return 1
            return None
        if self.min is not None and x < self.min:
            return self.min
        max_low = self.cluster[self.high(x)].maximum()
        if max_low is not None and self.low(x) < max_low:
            offset = self.cluster[self.high(x)].successor(self.low(x))
            return self.index(self.high(x), offset)
        succ_cluster = self.summary.successor(self.high(x))
        if succ_cluster is None:
            return None
        offset = self.cluster[succ_cluster].minimum()
        return self.index(succ_cluster, offset)

    def predecessor(self, x):
        if self.is_leaf():
            if x == 1 and self.min == 0:
                return 0
            return None
        if self.max is not None and x > self.max:
            return self.max
        min_low = self.cluster[self.high(x)].minimum()
        if min_low is not None and self.low(x) > min_low:
            offset = self.cluster[self.high(x)].predecessor(self.low(x))
            return self.index(self.high(x), offset)
        pred_cluster = self.summary.predecessor(self.high(x))
        if pred_cluster is None:
            if self.min is not None and x > self.min:
                return self.min
            return None
        offset = self.cluster[pred_cluster].maximum()
        return self.index(pred_cluster, offset)

    def insert_empty(self, x, n):
        self.min = x
        self.max = x
        self.min_cnt = self.max_cnt = n

    def insert(self, x, n=1):
        if self.min is None:
            self.insert_empty(x, n)
            return
        if x == self.max:
            self.max_cnt += n
        if x == self.min:
            self.min_cnt += n
            return
        if x < self.min:
            x, self.min = self.min, x
            n, self.min_cnt = self.min_cnt, n
        if not self.is_leaf():
            if self.cluster[self.high(x)].minimum() is None:
                self.summary.insert(self.high(x))
                self.cluster[self.high(x)].insert_empty(self.low(x), n)
            else:
                self.cluster[self.high(x)].insert(self.low(x), n)
        if x > self.max:
            self.max = x
            self.max_cnt = n

    def delete(self, x, n=1):
        if self.min == self.max:
            if self.min is None or self.min_cnt == n:
                self.min = self.max = None
                self.min_cnt = 0
            else:
                self.min_cnt -= n
            self.max_cnt = self.min_cnt
            return
        if self.is_leaf():
            if x == 0:
                self.min_cnt -= n
                if self.min_cnt == 0:
                    self.min = 1
                    self.min_cnt = self.max_cnt
            else:
                self.max_cnt -= n
                if self.max_cnt == 0:
                    self.max = 0
                    self.max_cnt = self.min_cnt
            return
        next_n = n
        if x == self.min:
            if self.min_cnt > n:
                self.min_cnt -= n
                return
            first_cluster = self.summary.minimum()
            x = self.index(first_cluster,
                           self.cluster[first_cluster].minimum())
            self.min = x
            self.min_cnt = self.cluster[first_cluster].min_cnt
            next_n = self.cluster[first_cluster].min_cnt
        self.cluster[self.high(x)].delete(self.low(x), next_n)
        if self.cluster[self.high(x)].minimum() is None:
            self.summary.delete(self.high(x))
            if x == self.max:
                if self.max == self.min:
                    self.max_cnt = self.min_cnt
                    return
                self.max_cnt -= n
                if self.max_cnt == 0:
                    sum_max = self.summary.maximum()
                    if sum_max is None:
                        self.max = self.min
                        self.max_cnt = self.min_cnt
                    else:
                        self.max = self.index(sum_max,
                                              self.cluster[sum_max].maximum())
                        self.max_cnt = self.cluster[sum_max].max_cnt
        elif x == self.max:
            if self.max == self.min:
                self.max_cnt = self.min_cnt
                return
            self.max_cnt -= n
            if self.max_cnt == 0:
                self.max = self.index(self.high(x),
                                      self.cluster[self.high(x)].maximum())
                self.max_cnt = self.cluster[self.high(x)].max_cnt

    def display(self, space=0, summary=False):
        disp = ' ' * space
        if summary:
            disp += 'S '
        else:
            disp += 'C '
        disp += str(self.u) + ' ' + str(self.min) + ' ' + str(self.max) + ' | '
        disp += str(self.min_cnt) + ' ' + str(self.max_cnt)
        print(disp)
        if not self.is_leaf():
            self.summary.display(space + 2, True)
            for c in self.cluster:
                c.display(space + 2)
```

### 20.3-2

> Modify vEB trees to support keys that have associated satellite data.

```python
class VanEmdeBoasTree:
    def __init__(self, u):
        self.u = u
        temp = u
        bit_num = -1
        while temp > 0:
            temp >>= 1
            bit_num += 1
        self.sqrt_h = 1 << ((bit_num + 1) // 2)
        self.sqrt_l = 1 << (bit_num // 2)
        self.min = None
        self.max = None
        self.min_data = None
        self.max_data = None
        if not self.is_leaf():
            self.summary = VanEmdeBoasTree(self.sqrt_h)
            self.cluster = []
            for _ in xrange(self.sqrt_h):
                self.cluster.append(VanEmdeBoasTree(self.sqrt_l))

    def is_leaf(self):
        return self.u == 2

    def high(self, x):
        return x / self.sqrt_l

    def low(self, x):
        return x % self.sqrt_l

    def index(self, x, y):
        return x * self.sqrt_l + y

    def minimum(self):
        return self.min

    def maximum(self):
        return self.max

    def member(self, x):
        if x == self.min or x == self.max:
            return True
        if self.is_leaf():
            return False
        return self.member(self.cluster[self.high(x)], self.low(x))

    def get_data(self, x):
        if x == self.min:
            return self.min_data
        if x == self.max:
            return self.max_data
        if self.is_leaf():
            return None
        return self.cluster[self.high(x)].get_data(self.low(x))

    def successor(self, x):
        if self.is_leaf():
            if x == 0 and self.max == 1:
                return 1
            return None
        if self.min is not None and x < self.min:
            return self.min
        max_low = self.cluster[self.high(x)].maximum()
        if max_low is not None and self.low(x) < max_low:
            offset = self.cluster[self.high(x)].successor(self.low(x))
            return self.index(self.high(x), offset)
        succ_cluster = self.summary.successor(self.high(x))
        if succ_cluster is None:
            return None
        offset = self.cluster[succ_cluster].minimum()
        return self.index(succ_cluster, offset)

    def predecessor(self, x):
        if self.is_leaf():
            if x == 1 and self.min == 0:
                return 0
            return None
        if self.max is not None and x > self.max:
            return self.max
        min_low = self.cluster[self.high(x)].minimum()
        if min_low is not None and self.low(x) > min_low:
            offset = self.cluster[self.high(x)].predecessor(self.low(x))
            return self.index(self.high(x), offset)
        pred_cluster = self.summary.predecessor(self.high(x))
        if pred_cluster is None:
            if self.min is not None and x > self.min:
                return self.min
            return None
        offset = self.cluster[pred_cluster].maximum()
        return self.index(pred_cluster, offset)

    def insert_empty(self, x, data):
        self.min = x
        self.max = x
        self.min_data = self.max_data = data

    def insert(self, x, data):
        if self.min is None:
            self.insert_empty(x, data)
        else:
            if x < self.min:
                x, self.min = self.min, x
                data, self.min_data = self.min_data, data
            if not self.is_leaf():
                if self.cluster[self.high(x)].minimum() is None:
                    self.summary.insert(self.high(x), data)
                    self.cluster[self.high(x)].insert_empty(self.low(x), data)
                else:
                    self.cluster[self.high(x)].insert(self.low(x), data)
            if x > self.max:
                self.max = x
                self.max_data = data

    def delete(self, x):
        if self.min == self.max:
            self.min = self.max = None
            self.min_data = self.max_data = None
        elif self.is_leaf():
            if x == 0:
                self.min = 1
                self.min_data = self.max_data
            else:
                self.min = 0
            self.max = self.min
            self.max_data = self.min_data
        else:
            if x == self.min:
                first_cluster = self.summary.minimum()
                x = self.index(first_cluster,
                               self.cluster[first_cluster].minimum())
                self.min = x
                self.min_data = self.cluster[first_cluster].min_data
            self.cluster[self.high(x)].delete(self.low(x))
            if self.cluster[self.high(x)].minimum() is None:
                self.summary.delete(self.high(x))
                if x == self.max:
                    sum_max = self.summary.maximum()
                    if sum_max is None:
                        self.max = self.min
                        self.max_data = self.min_data
                    else:
                        self.max = self.index(sum_max,
                                              self.cluster[sum_max].maximum())
                        self.max_data = self.cluster[sum_max].max_data
            elif x == self.max:
                self.max = self.index(self.high(x),
                                      self.cluster[self.high(x)].maximum())
                self.max_data = self.cluster[self.high(x)].max_data

    def display(self, space=0, summary=False):
        disp = ' ' * space
        if summary:
            disp += 'S '
        else:
            disp += 'C '
        disp += str(self.u) + ' ' + str(self.min) + ' ' + str(self.max)
        print(disp)
        if not self.is_leaf():
            self.summary.display(space + 2, True)
            for c in self.cluster:
                c.display(space + 2)
```

### 20.3-3

> Write pseudocode for a procedure that creates an empty van Emde Boas tree.

See exercise 20.3-1 and exercise 20.3-2.