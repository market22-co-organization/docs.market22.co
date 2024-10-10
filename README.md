# Market22 Public API Documentation

## Base URL

All API requests should be made to the following base URL:

```
https://api.market22.co/api
```

## Authentication

To authenticate API requests, you need an API key. You can generate an API key by navigating to:

- **Settings > API Keys** in your dashboard, or
- Directly via [https://www.market22.co/dashboard/settings?tab=api-keys](https://www.market22.co/dashboard/settings?tab=api-keys).

Include the API key in your request headers as follows:

```
Authorization: Bearer YOUR_API_KEY
```

## Endpoints

### Products

#### Fetch All Products

- **Endpoint**: `GET /products`
- **Description**: Retrieve a list of all products associated with the authenticated account.
- **Response**:
  - Success: `200 OK` with a list of products.
  - Error: Appropriate error message with status code.

#### Create a Product

- **Endpoint**: `POST /products`
- **Description**: Create a new product.
- **Request Body**:
  - `name`: string (required)
  - `description`: string (required)
  - `thumbnail`: string (URI, required)
  - `settings`: object (required)
    - `type`: "digital" (default, required)
    - `category`: "saas" (required)
    - `pricing_type`: "onetime" | "recurring" (required)
    - `price`: object (conditional)
      - For "onetime":
        - `default`: number (min 1, required)
        - `is_dynamic`: boolean (required)
        - `striked_out_original_price`: object (optional)
          - `enabled`: boolean (required)
          - `value`: number (required)
      - For "recurring":
        - `variations`: array of objects (required)
          - `value`: number (min 1, required)
          - `interval`: string (valid intervals, required)
    - `commission`: object (required)
      - `default`: number (min 1, max 100, required)
      - `variations`: array of objects
        - `value`: number (min 1, max 100, required)
        - `email`: string (email, required)
    - `discounts`: array of discount objects (optional)
    - `shareholders`: array of shareholder objects (optional)
    - `upsells`: array of strings (optional)
    - `public`: boolean (required)
  - `resources`: object
    - `links`: object
      - `sales_page`: string (optional)
      - `affiliate_resources_url`: string (optional)
      - `access_url`: string (required)
    - `webhook`: object
      - `url`: string (required)
      - `secret`: string (required)
- **Response**:
  - Success: `201 Created` with the created product details.
  - Error: Appropriate error message with status code.

#### Update a Product

- **Endpoint**: `PATCH /products/:id`
- **Description**: Update an existing product.
- **Request Parameters**:
  - `id`: string (Product ID)
- **Request Body**: Similar to create product schema with the same validation rules.
- **Response**:
  - Success: `200 OK` with updated product details.
  - Error: Appropriate error message with status code.

#### Fetch a Single Product

- **Endpoint**: `GET /products/:id`
- **Description**: Retrieve details of a specific product.
- **Request Parameters**:
  - `id`: string (Product ID)
- **Response**:
  - Success: `200 OK` with product details.
  - Error: `404 Not Found` if product does not exist.

#### Fetch Product Orders

- **Endpoint**: `GET /products/:id/orders`
- **Description**: Retrieve orders associated with a specific product.
- **Request Parameters**:
  - `id`: string (Product ID)
- **Response**:
  - Success: `200 OK` with a list of orders.
  - Error: `404 Not Found` if product does not exist.

### Orders

#### Fetch All Orders

- **Endpoint**: `GET /orders`
- **Description**: Retrieve a list of all orders associated with the authenticated account.
- **Response**:
  - Success: `200 OK` with a list of orders.
  - Error: Appropriate error message with status code.

#### Fetch a Single Order

- **Endpoint**: `GET /orders/:id`
- **Description**: Retrieve details of a specific order.
- **Request Parameters**:
  - `id`: string (Order ID or Reference)
- **Response**:
  - Success: `200 OK` with order details.
  - Error: `404 Not Found` if order does not exist.

## Interface Breakdown

### Product Interface

- **\_id**: string (Unique identifier for the product)
- **name**: string (Name of the product)
- **slug**: string (URL-friendly version of the product name)
- **description**: string (Detailed description of the product)
- **thumbnail**: string (URL to the product's thumbnail image)
- **account**: Account (Account associated with the product)
- **approved_by**: Account (optional, Account that approved the product)
- **deactivated_by**: Account (optional, Account that deactivated the product)
- **deactivation_message**: string (optional, Message explaining why the product was deactivated)
- **settings**: object (Configuration settings for the product)
  - **type**: "digital" (Type of product, e.g., digital)
  - **category**: "course" | "saas" | "file" (Category of the product)
  - **pricing_type**: "onetime" | "recurring" (Pricing model for the product)
  - **price**: object (Pricing details for the product)
    - **default**: string (Default price of the product)
    - **is_dynamic**: boolean (Indicates if the price is dynamic)
    - **striked_out_original_price**: object (Original price before any discount)
      - **enabled**: boolean (Indicates if the original price is shown)
      - **value**: string (Value of the original price)
    - **variations**: PriceVariation[] (Array of price variations)
  - **commission**: object (Commission details for the product)
    - **default**: string (Default commission percentage)
    - **variations**: array of objects (Array of commission variations)
      - **email**: string (Email associated with the commission variation)
      - **value**: string (Value of the commission variation)
  - **discounts**: Discount[] (Array of discounts available for the product)
  - **shareholders**: ShareHolder[] (Array of shareholders for the product)
  - **upsells**: Product[] | string[] (Array of upsell products or their IDs)
  - **public**: boolean (Indicates if the product is public)
- **resources**: object (Resources associated with the product)
  - **links**: object (Links related to the product)
    - **sales_page**: string (optional, URL to the sales page)
    - **access_url**: string (URL to access the product)
    - **affiliate_resources_url**: string (optional, URL to affiliate resources)
  - **webhook**: object (optional, Webhook details)
    - **url**: string (optional, URL of the webhook)
    - **secret**: string (optional, Secret key for the webhook)
- **status**: "active" | "inactive" | "pending" (Current status of the product)
- **createdAt**: Date (Date when the product was created)

### Order Interface

- **\_id**: string (Unique identifier for the order)
- **product**: Product | any (Product associated with the order)
- **customer**: Account | any (Customer who placed the order)
- **applied_discount**: object (Discount applied to the order)
  - **code**: string (Discount code)
  - **percentage**: string (Discount percentage)
- **total_charged**: number (Total amount charged for the order)
- **fee**: number (Fee associated with the order)
- **pricing_type**: "recurring" | "onetime" (Pricing model for the order)
- **participants**: OrderParticipant[] (Array of participants in the order)
- **payment_provider**: "flutterwave" | "stripe" | "cryptomus" (Payment provider used for the order)
- **status**: "successful" | "abandoned" | "processing" (Current status of the order)
- **createdAt**: Date (Date when the order was created)
- **reference**: string (Reference ID for the order)
- **charge_currency**: string (Currency used for the charge)
- **paid_at**: string | Date (Date when the order was paid)
- **paid**: boolean (Indicates if the order has been paid)
- **payment_information**: object (Payment information for the order)
  - **payment_method**: string (Payment method used)
  - **payment_method_details**: Card | null | {} (Details of the payment method)
- **subscription**: object (optional, Subscription details)
  - **last_billed_at**: string | Date (Date of the last billing)
  - **cancel_at_period_end**: boolean (Indicates if the subscription will be canceled at the end of the period)
  - **status**: "active" | "cancelled" | "awaiting_payment" (Current status of the subscription)
  - **selected_plan**: string (Selected plan for the subscription)

### Webhook Events Overview

Market22 uses Webhooks to notify your internal systems about significant changes in order status. These events are public-facing and are intended for integration with other services.

### Authentication

To ensure secure communication between Market22 and your application, it is essential to verify webhook secrets. This guide provides developers with the necessary steps to implement webhook secret verification effectively.

## What is a Webhook Secret?

A webhook secret is a unique key that you provide when configuring a webhook for your product on Market22. It is used to authenticate incoming webhook requests from Market22, ensuring that they are legitimate and have not been tampered with.

## How to Verify Webhook Secrets

### Step 1: Provide a Webhook Secret

When you create or configure a product on Market22, you will be asked to provide a webhook secret. This secret will be used to verify the authenticity of incoming webhook requests.

### Step 2: Receive the Webhook Request

When Market22 sends a webhook event to your endpoint, it includes the secret in the request headers. Specifically, the header `x-market22-webhook-secret` contains the secret key.

### Step 3: Compare the Secrets

Upon receiving a webhook request, your application should:

1. Extract the `x-market22-webhook-secret` from the request headers.
2. Compare this secret with the expected secret you provided during product configuration.

### Example Verification Code

Here is a basic example of how you might implement webhook secret verification in a Node.js application:

```typescript
const express = require("express");
const app = express();
app.post("/webhook-endpoint", (req, res) => {
  const receivedSecret = req.headers["x-market22-webhook-secret"];
  const expectedSecret = process.env.MARKET22_WEBHOOK_SECRET; // Store your secret securely
  if (receivedSecret === expectedSecret) {
    // Process the webhook event
    res.status(200).send("Webhook verified and processed");
  } else {
    // Reject the request
    res.status(401).send("Unauthorized: Invalid webhook secret");
  }
});
app.listen(3000, () => {
  console.log("Server is running on port 3000");
});
```

### Step 4: Handle Verification Failures

If the secrets do not match, reject the request and log the attempt for further investigation. This helps in identifying any unauthorized access attempts.

## Best Practices for Webhook Secret Management

- **Store Secrets Securely**: Use environment variables or secure vaults to store your webhook secrets.
- **Use Strong Secrets**: Generate strong, random secrets to enhance security.

By following these steps and best practices, you can ensure that your application securely processes webhook events from Market22, maintaining the integrity and security of your data.

### Key Webhook Events

Here are the primary webhook events related to orders:

1. **Product Approved (`product.approved`)**: Triggered when a product is approved. This event can be used to initiate any setup or provisioning processes in your internal systems.

2. **Product Updated (`product.updated`)**: Triggered when a product is updated.

3. **Product Deactivated (`product.deactivated`)**: Triggered when a product is deactivated.

4. **Order Created (`order.created`)**: Triggered when a new order is created. This event can be used to initiate any setup or provisioning processes required for the order.

5. **Order Paid (`order.paid`)**: Triggered when an order's payment is successfully processed. This event is crucial for confirming the transaction and possibly granting access to the purchased product or service.

6. **Order Abandoned (`order.abandoned`)**: Triggered when an order is left incomplete or not paid. This can be used to send reminders or follow-up communications to the customer.

7. **Subscription Renewed (`order.subscription_renewed`)**: Triggered when a subscription is successfully renewed. This event can be used to update billing cycles or notify the customer of the renewal.

8. **Subscription Payment Failed (`order.subscription_payment_failed`)**: Triggered when a payment for a subscription renewal fails. This can be used to alert the customer and prompt them to update their payment information.

9. **Subscription Updated (`order.subscription_updated`)**: Triggered when a subscription's plan or details are changed. This event can be used to adjust billing or service levels accordingly.

10. **Subscription Cancelled (`order.subscription_cancelled`)**: Triggered when a subscription is cancelled. This event can be used to stop any recurring services or notify the customer of the cancellation.

### Handling Webhook Events

When your system receives these webhook events, it is recommended to:

- **Pull Order Details**: Use the `orderId` provided in the event payload to fetch the latest details of the order. This ensures that you have the most up-to-date information about the order status and any associated data.

- **Process Accordingly**: Based on the event type, perform the necessary actions such as updating records, notifying users, or integrating with other systems.

By handling these events effectively, you can ensure that your system remains synchronized with the order lifecycle and provides a seamless experience for your users.

### Checkout Shortcuts

Market22 provides several query parameters that can be used to customize the checkout experience for users. These parameters help in pre-selecting subscription plans, pre-filling user information, and managing subscriptions effectively. Importantly, using these parameters creates a browser session, ensuring data persistence across page refreshes.

#### 1. Pre-selecting a Subscription Plan

- **Price Query Parameter**: By including a `price` query parameter in the checkout URL, you can pre-select a specific plan for recurring subscriptions. This is particularly useful when you want to direct users to a specific pricing tier without requiring them to manually select it during checkout.

  **Example Usage**:

  ```
  https://www.market22.co/checkout/:id/pay?price=plan_id
  ```

  This URL will automatically select the plan associated with `plan_id` for the user, streamlining the checkout process. A session is created, ensuring that the selected plan persists across page refreshes.

#### 2. Pre-filling Email Address

- **Email Query Parameter**: Including an `email` query parameter in the checkout URL allows you to pre-fill the email address field. This is beneficial for users who are redirected from your application, as it provides a seamless transition and reduces the need for manual input.

  **Example Usage**:

  ```
  https://www.market22.co/checkout/:id/pay?email=user@example.com
  ```

  When this parameter is used, a session is created, which means that the data persists across page refreshes. This gives the user a session-like experience, as if they were continuously interacting with your application. It is particularly useful if you want to enforce the use of a specific email address for the checkout process.

#### 3. Managing Subscriptions

- **Subscription Management via Email**: To manage an existing subscription, you can direct users to the checkout page with their email address included in the query parameter. If the email corresponds to a successful order, the user will be prompted to manage their subscription.

  **Example Usage**:

  ```
  https://www.market22.co/checkout/:id/pay?email=user@example.com
  ```

  This approach allows users to easily access their subscription management options, such as updating their plan or cancelling the subscription, by simply providing their email address. It ensures that users can manage their subscriptions without needing to navigate through multiple pages or remember additional login credentials.

### Benefits of Checkout Shortcuts

- **Streamlined User Experience**: By pre-selecting plans and pre-filling user information, you reduce friction in the checkout process, leading to higher conversion rates and improved user satisfaction.

- **Consistency Across Sessions**: The use of browser sessions ensures that user data is retained across page refreshes, providing a consistent and uninterrupted experience.

- **Simplified Subscription Management**: Allowing users to manage subscriptions directly from the checkout page with minimal input simplifies the process and enhances user control over their subscriptions.

By leveraging these checkout shortcuts, you can create a more intuitive and efficient checkout process that aligns with user expectations and business goals.
