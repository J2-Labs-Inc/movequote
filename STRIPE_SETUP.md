# Stripe Setup Instructions for CleanlyQuote

## Overview
This guide walks you through setting up Stripe payments for CleanlyQuote. The backend API is already configured to handle Stripe webhooks and subscriptions - you just need to add the API keys.

## Step 1: Create Stripe Product & Price

1. Go to https://dashboard.stripe.com/products
2. Click **"Add product"**
3. Fill in details:
   - **Name:** CleanlyQuote Professional
   - **Description:** Unlimited cleaning estimates and premium features
   - **Pricing:**
     - **Price:** $29.00
     - **Billing period:** Monthly
     - **Currency:** USD
4. Click **"Save product"**
5. **Copy the Price ID** - it starts with `price_` (e.g., `price_1A2B3C4D5E6F7G8H`)

## Step 2: Get API Keys

1. Go to https://dashboard.stripe.com/apikeys
2. You'll see two keys:
   - **Publishable key** (starts with `pk_test_` or `pk_live_`)
   - **Secret key** (starts with `sk_test_` or `sk_live_`)
3. Copy both keys (you'll need them in steps 3 and 5)

**Note:** Use test keys (`pk_test_` / `sk_test_`) for development, live keys for production.

## Step 3: Configure Railway Backend Environment Variables

1. Go to https://railway.app
2. Navigate to your **cleanlyquote** project → **api** service
3. Click **"Variables"** tab
4. Add the following environment variables:

```env
STRIPE_SECRET_KEY=sk_test_... (or sk_live_... for production)
STRIPE_PRICE_ID=price_... (from Step 1)
```

5. Click **"Add"** for each variable
6. Railway will automatically redeploy the API with these new variables

## Step 4: Set Up Stripe Webhook

1. Go to https://dashboard.stripe.com/webhooks
2. Click **"Add endpoint"**
3. Fill in details:
   - **Endpoint URL:** `https://api-production-9bf9.up.railway.app/api/webhook`
   - **Description:** CleanlyQuote Subscription Events
   - **Events to send:**
     - `checkout.session.completed`
     - `customer.subscription.created`
     - `customer.subscription.updated`
     - `customer.subscription.deleted`
     - `invoice.payment_succeeded`
     - `invoice.payment_failed`
4. Click **"Add endpoint"**
5. Click on the newly created webhook to view details
6. **Copy the "Signing secret"** (starts with `whsec_`)

## Step 5: Add Webhook Secret to Railway

1. Back in Railway → **cleanlyquote** project → **api** service → **Variables**
2. Add:
```env
STRIPE_WEBHOOK_SECRET=whsec_... (from Step 4)
```
3. Railway will redeploy again

## Step 6: Update Frontend with Publishable Key

### Option A: Manual Edit (Quick)
1. Go to your **cleanlyquote-old** repository
2. Edit `public/app.html`
3. Find line 2554 (search for `pk_test_your_publishable_key_here`)
4. Replace with your actual publishable key:
```javascript
const stripe = Stripe('pk_test_... OR pk_live_...'); // Your key from Step 2
```
5. Uncomment line 2549 and 2551 (remove `<!--` and `-->` around Stripe.js script)
6. Commit and push - Railway will auto-deploy

### Option B: Environment Variable (Better)
If you want to avoid hardcoding, set up environment variable replacement in your build process.

## Step 7: Test the Integration

### Test Signup Flow
1. Go to https://getcleanlyquote.com/auth?mode=signup
2. Create a test account
3. Go to https://getcleanlyquote.com/app
4. Click **"Upgrade to Pro"**
5. Should redirect to Stripe checkout

### Test Checkout
1. Use Stripe test card: **4242 4242 4242 4242**
   - Any future expiry date
   - Any 3-digit CVC
   - Any postal code
2. Complete checkout
3. You should be redirected back to the app
4. Your subscription status should now show "active"

### Verify Webhook
1. Go back to Stripe Dashboard → Webhooks
2. Click on your webhook endpoint
3. Check the **"Recent events"** tab
4. You should see events like `checkout.session.completed` with status **200 OK**

## Step 8: Go Live (When Ready)

1. In Stripe Dashboard, toggle from **Test mode** to **Live mode**
2. Repeat Steps 2-6 with **live keys** (start with `sk_live_` and `pk_live_`)
3. Update Railway production variables
4. Test with a real card (you can refund it immediately)

## Troubleshooting

### "Webhook signature verification failed"
- Double-check `STRIPE_WEBHOOK_SECRET` matches the signing secret from Stripe Dashboard
- Make sure you're sending events to the correct environment (test/live)

### "No checkout URL returned"
- Check Railway logs for API errors
- Verify `STRIPE_SECRET_KEY` and `STRIPE_PRICE_ID` are set correctly
- Make sure Price ID matches the product you created

### Subscription status not updating
- Check webhook events in Stripe Dashboard
- Verify webhook endpoint URL is correct
- Check Railway API logs for webhook processing errors

## Current Workaround

**Note:** There's already a working Stripe payment link hardcoded in the frontend:
```
https://buy.stripe.com/5kQ5kF91QcJhaI250I1sQ00
```

This works but doesn't integrate with your backend API. Completing the above steps will enable full API integration with automatic subscription status updates.

## Need Help?

- **Stripe Documentation:** https://stripe.com/docs/billing/subscriptions/overview
- **Railway Support:** https://railway.app/help
- **Test Cards:** https://stripe.com/docs/testing

## Security Notes

- **Never commit API keys to Git** - always use environment variables
- Use test keys for development, live keys only in production
- Rotate keys if they're ever exposed
- Enable Stripe Radar for fraud prevention
