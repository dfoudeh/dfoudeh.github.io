+++
title = 'smileyCTF 2025 - web/dry-ice-n-co'
date = 2025-06-16T20:18:36-04:00
draft = false
+++

![dryice1](/images/dryice1.png)


We are given a Java Spring Boot web application. The website sells dry ice - we can purchase a flag that costs $1,000,000 but we only start with $100. The first vulnerabilty is with the admin check:

```java
if ((user.admin = true) && user != null && name != "flag") {
    availableProducts.add(new DryIceProduct(name, price, description));
}
```

This uses assignement `=` instead of comparison `==` so user.admin is set to true for any user making a request.

Next, the shopping cart total calculatoin has an integer overflow 

```java
public int getTotal() {
    int total = items.stream()
        .mapToInt(CartItem::getTotal)
        .sum();
    total = Math.abs(total); 
    
    if (isCouponValid()) {
        total = (int)(total * (100.0 - (double)DISCOUNT_PERCENTAGE) / 100.0);
    }
    
    return total;
}
```

Cart item getTotal:
```java
public int getTotal() {
    return Math.abs(price) * Math.abs(quantity);
}
```

Negative purchase addition:
```java
public void purchase() {
    if (canAfford()) {
        balance -= getTotal();
        // ...
    }
}
```

As admin, we can create new items to be sold in the shop. If we create an item with the value `-1073741824` then `Math.abs(-1073741824) * 2 = 2,147,483,648` This is equal to 2^31 which overflows to -2,147,483,648 (Integer.MIN_VALUE)
Math.abs(Integer.MIN_VALUE) = -2,147,483,648 (remains negative)

So when we buy these items, our total balance is set to -2,147,483,648.

Now we can apply our coupon code SMILEICE for a 20% discount. The negative total becomes: -2,147,483,648 × 0.8 = -1,717,986,918 When we make a purchase with the negative total the balacne is calculated as 100 - (-1,717,986,918) = 1,717,987,018. And we have enough points to buy our flag


```.;,;.{this_is_not_a_political_statement_btw}```