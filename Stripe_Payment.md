# Stripe Payment Interagtion Made Easy

## How stripe Work
![](https://beta.appflowy.cloud/api/file_storage/a8ed4b92-e4ef-46cf-b224-51bceb269836/blob/6073211808283955384.png)

## Frontend Implementation
```javascript
// Payement Stripe Integration
  const makePaymentStripe = async () => {
    try {
      const response = await fetch(`${apiUrl}/create-checkout-session`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          products: [
            {
              id: 1,
              dish: "punjabi",
              imgdata:
                "https://b.zmtcdn.com/data/pictures/9/18857339/8f53919f1175c08cf0f0371b73704f9b_o2_featured_v2.jpg?output-format=webp",
              address: "North Indian, Biryani, Mughlai",
              delimg:
                "https://b.zmtcdn.com/data/o2_assets/0b07ef18234c6fdf9365ad1c274ae0631612687510.png?output-format=webp",
              somedata: " 1175 + order placed from here recently",
              price: 350,
              rating: "3.8",
              arrimg:
                "https://b.zmtcdn.com/data/o2_assets/4bf016f32f05d26242cea342f30d47a31595763089.png?output-format=webp",
              qnty: 1,
            },
          ],
        }),
      });

      const session = await response.json();

      const stripe = await loadStripe(publishable_key);
      const result = await stripe.redirectToCheckout({
        sessionId: session.id,
      });

      if (result.error) {
        console.error(result.error);
      }
    } catch (error) {
      console.error("Error creating checkout session:", error);
    }
  };
```
> 📌
> Make sure The Cart Array Should be in this Format 

```javascript
 [
            {
              id: 1,
              dish: "punjabi",
              imgdata:
                "https://b.zmtcdn.com/data/pictures/9/18857339/8f53919f1175c08cf0f0371b73704f9b_o2_featured_v2.jpg?output-format=webp",
              address: "North Indian, Biryani, Mughlai",
              delimg:
                "https://b.zmtcdn.com/data/o2_assets/0b07ef18234c6fdf9365ad1c274ae0631612687510.png?output-format=webp",
              somedata: " 1175 + order placed from here recently",
              price: 350,
              rating: "3.8",
              arrimg:
                "https://b.zmtcdn.com/data/o2_assets/4bf016f32f05d26242cea342f30d47a31595763089.png?output-format=webp",
              qnty: 1,
            },
],
```

### Frontend ENV
```
VITE_PUBLISHABLE_KEY
```

## Backend Implementation

### STEP 1 : CREATE A SESSION
```javascript
app.post("/create-checkout-session", async (req, res) => {
  const { products } = req.body;

  const lineItems = products.map((product) => ({
    price_data: {
      currency: "inr",
      product_data: {
        name: product.dish,
        images: [product.imgdata],
      },
     unit_amount: Math.round(product.price * 100),
    },
    quantity: product.qnty,
  }));

  console.log(lineItems)

  const session = await stripe.checkout.sessions.create({
    payment_method_types: ["card"],
    line_items: lineItems,
    mode: "payment",
    success_url: "http://localhost:3000/success",
    cancel_url: "http://localhost:3000/cancel",
  });

  res.json({ id: session.id });
});
```
> 📌
> Make Sure our Frontend will has this both Routes 

```markdown
success Page : http://localhost:3000/success
cancel Page : http://localhost:3000/cancel
```

### STEP 2 : Create a Webhook to Update the successfull Payement in Database
```javascript
app.post('stripe/webhook', express.raw({type: 'application/json'}), (request, response) => {
  const sig = request.headers['stripe-signature'];

  let event;

  try {
    event = stripe.webhooks.constructEvent(request.body, sig, process.env.STRIPE_SECRET);
  } catch (err) {
    response.status(400).send(`Webhook Error: ${err.message}`);
    return;
  }

  // Handle the event
  console.log(`Unhandled event type ${event.type}`);
// Need to Store it database

  // Return a 200 response to acknowledge receipt of the event
  response.send();
});
```

### **Backend ENV**
```
STRIPE_SECRET
```