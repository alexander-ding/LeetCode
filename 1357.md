# [1357. Apply Discount Every n Orders](https://leetcode.com/problems/apply-discount-every-n-orders/)

There is a sale in a supermarket, there will be a `discount` every `n` customer.

There are some products in the supermarket where the id of the `i-th` product is `products[i]` and the price per unit of this product is `prices[i]`.

The system will count the number of customers and when the `n-th` customer arrive he/she will have a `discount` on the bill. (i.e if the cost is `x` the new cost is `x - (discount * x) / 100)`. Then the system will start counting customers again.

The customer orders a certain amount of each product where `product[i]` is the id of the `i-th` product the customer ordered and `amount[i]` is the number of units the customer ordered of that product.

Implement the `Cashier` class:

- `Cashier(int n, int discount, int[] products, int[] prices)` Initializes the object with `n`, the discount, the products and their prices.
- `double getBill(int[] product, int[] amount)` returns the value of the bill and apply the discount if needed. Answers within `10^-5` of the actual value will be accepted as correct.

**Example 1:**
```
Input
["Cashier","getBill","getBill","getBill","getBill","getBill","getBill","getBill"]
[[3,50,[1,2,3,4,5,6,7],[100,200,300,400,300,200,100]],[[1,2],[1,2]],[[3,7],[10,10]],[[1,2,3,4,5,6,7],[1,1,1,1,1,1,1]],[[4],[10]],[[7,3],[10,10]],[[7,5,3,1,6,4,2],[10,10,10,9,9,9,7]],[[2,3,5],[5,3,2]]]
Output
[null,500.0,4000.0,800.0,4000.0,4000.0,7350.0,2500.0]
Explanation
Cashier cashier = new Cashier(3,50,[1,2,3,4,5,6,7],[100,200,300,400,300,200,100]);
cashier.getBill([1,2],[1,2]);                        // return 500.0, bill = 1 * 100 + 2 * 200 = 500.
cashier.getBill([3,7],[10,10]);                      // return 4000.0
cashier.getBill([1,2,3,4,5,6,7],[1,1,1,1,1,1,1]);    // return 800.0, The bill was 1600.0 but as this is the third customer, he has a discount of 50% which means his bill is only 1600 - 1600 * (50 / 100) = 800.
cashier.getBill([4],[10]);                           // return 4000.0
cashier.getBill([7,3],[10,10]);                      // return 4000.0
cashier.getBill([7,5,3,1,6,4,2],[10,10,10,9,9,9,7]); // return 7350.0, Bill was 14700.0 but as the system counted three more customers, he will have a 50% discount and the bill becomes 7350.0
cashier.getBill([2,3,5],[5,3,2]);                    // return 2500.0
```

**Constraints:**

- `1 <= n <= 10^4`
- `0 <= discount <= 100`
- `1 <= products.length <= 200`
- `1 <= products[i] <= 200`
- There are **not** repeated elements in the array products.
- `prices.length == products.length`
- `1 <= prices[i] <= 1000`
- `1 <= product.length <= products.length`
- `product[i]` exists in `products`.
- `amount.length == product.length`
- `1 <= amount[i] <= 1000`
- At most `1000` calls will be made to `getBill`.
- Answers within `10^-5` of the actual value will be accepted as correct.

## Solution
### Approach 1: Hashtable Lookup
#### Intuition
This problem can be broken down into two independent components. The first challenge is to compute the total bill for a customer, and the second is to periodically apply a discount to the total bill. 

For the first challenge, note that in `getBill`, we are given a list of product IDs and will need to retrieve their corresponding prices to compute the total bill. To efficiently do so, we can combine the information in `products` and `prices` to create a lookup table `priceTable` that maps each product ID to its price. In Python, we can use the built-in `Dict`. `priceTable` only needs to be created once as the `Cashier` object is initialized, and it can be reused later whenever `getBill` is called. Now, we can compute `totalBill` as the sum of `priceTable[product[i]] * amount[i]` for `0 <= i < product.length`. 

For the second challenge, we can simply maintain an internal `counter` to keep track of the number of customers processed. Every time `getBill` is called, we increment `counter` by 1. If `counter` indicates that the current customer is the `n-th` customer, we apply the discount to the computed total bill and reset the counter. 

As a small optimization, note that `x - (discount * x) / 100)` can be rewritten as `x * ((100 - discount) / 100)`. Therefore, we can define `discountMultiplier = (100 - discount) / 100` and simply compute the discounted price as `x * discountMultiplier`. 

#### Algorithm
```Python
class Cashier:
    def __init__(self, n: int, discount: int, products: List[int], prices: List[int]):
        self.n = n
        self.discountMultiplier = (100 - discount) / 100 # compute the discount multiplier
        # create a lookup table mapping from product ID to price
        self.priceTable = {}
        for i, product in enumerate(products):
            self.priceTable[product] = prices[i]
        self.counter = 0
        
    def getBill(self, product: List[int], amount: List[int]) -> float:
        # compute total bill
        totalBill = 0.0
        for i, productID in enumerate(product):
             totalBill += self.priceTable[productID] * amount[i]
        
        # every n bills, apply discount
        if (self.counter == (self.n-1)):
            self.counter = 0
            return totalBill * self.discountMultiplier

        # otherwise, return bill as is
        self.counter += 1
        return totalBill
```

#### Complexity Analysis
- Time complexity: 
  - `__init__`: ![equation](http://latex.codecogs.com/svg.latex?%24O%28n%29%24), where `n` is `products.length`. We iterate through the `products` `List` and adds to a `Dict` in each iteration, which is ![equation](https://latex.codecogs.com/svg.latex?%24O%281%29%24)
  - `getBill`: ![equation](http://latex.codecogs.com/svg.latex?%24O%28n%29%24), where `n` is `product.length`. We iterate through the `product` `List`, and the `List` and `Dict` lookups in each iteration both have complexitiy ![equation](https://latex.codecogs.com/svg.latex?%24O%281%29%24). 
- Space complexity: 
  - `__init__`: ![equation](http://latex.codecogs.com/svg.latex?%24O%28n%29%24), where `n` is `products.length`. Besides the constant variables, we only create the size-`n` `Dict` `priceTable`, which has space complexitiy ![equation](http://latex.codecogs.com/svg.latex?%24O%28n%29%24). 
  - `getBill`: ![equation](http://latex.codecogs.com/svg.latex?%24O%28n%29%24). The only variable created is `totalBill`. 
