---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell


includes:
  - errors

search: true
---

# API reference

This is the public API documentation for FirstPromoter.

All our API calls are sent using the usual key-value pair parameters(see in the right), not JSON. The responses however are in JSON format.

# Authentication

```shell
curl -X POST "api_endpoint_here"
  -d "param1=value"
  -d "param2=value2"
  -H "x-api-key: {apikey}"
```

> Make sure to replace `{apikey}` with your API key.

FirstPromoter uses API key to authorize the API requests. The API key can be
found on your Settings page - login to FirstPromoter and click "Settings" button from the top bar.

FirstPromoter expects for the API key to be included in all API requests to the server in a header that looks like the following:

`x-api-key: {apikey}`

<aside class="notice">
You must replace <code>{apikey}</code> with your account API key.
</aside>

# Tracking API

Our tracking API allows companies to track any type of signups, sales, cancellations and refunds for any billing provider, you are not limited to our built-in integrations with Stripe, Chargebee, Recurly and Braintree.

In order to send a Tracking API call you'll need to pass the custom tracking integration ID(wid) as parameter with every call. Note: other API calls do not require this.

**To get your Integration ID(wid):**

1. click "Settings" button from the top bar
2. click on "Integrations" button
3. click on "Setup" button on the "Tracking API" integration

## Tracking leads and sign-ups

```shell
curl -X POST "https://firstpromoter.com/api/v1/track/signup"
  -d "wid=b850ac4wer56hy1b5ef41"
  -d "email=shelley@example.com"
  -d "first_name=Shelley"
  -d "uid=cbdemo_shelley"
  -d "tid=3491ff2f-7803-4467-8863-15b54frt8dyy"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "id": 1707,
  "type": "signup",
  "amount_cents": null,
  "lead": {
    "id": 943,
    "state": "signup",
    "first_name": "Shelley",
    "last_name": "",
    "email": "shelley@example.com",
    "uid": "cbdemo_shelley",
    "customer_since": null,
    "plan_name": null
    "suspicion": "no_suspicion"
  },
  "promoter": {
    "id": 1983,
    "cust_id": null,
    "email": "test@test.com",
    "temp_password": "u1PptB",
    "default_promotion_id": 1986,
    "default_ref_id": "test_ref_id",
    "earnings_balance": null,
    "current_balance": null,
    "paid_balance": null
  }
}
```

With this call you can track the referral signs-ups server-side. This is not for tracking the actual sales and commissions.

Sign-ups are tracked as leads in FirstPromoter so when a person referred by the promoter/affiliate signs up, a new lead should be added inside FirstPromoter (you can see them inside the "Leads" section).

The recommended way to do this is to grab the **\_fprom_track** cookie value(which keeps the tracking id and referral identification) on your server and send it along with the sign-up data through the **tid** parameter.

**_Alternative_**: In some special cases, you can refer sign ups directly to a promoter, by passing the referral id through **ref_id** parameter. Be careful using this because the referral id can be modified by the promoter by default, however you can disable that from the campaign configuration page.

### HTTP Request

`POST https://firstpromoter.com/api/v1/track/signup`

### Query Parameters

| Parameter  | Required                   | Description                                                                                                                                                                                                                                                                                                                                                                                                   |
| ---------- | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| wid        | yes                        | integration id. [How to get the it?](#tracking-api)                                                                                                                                                                                                                                                                                                                                                           |
| email      | yes                        | email of the lead/sign-up                                                                                                                                                                                                                                                                                                                                                                                     |
| uid        | no                         | id to match the sale with the lead if the email can be changed before the first sale. If the sales are tracked by our built-in integrations and not by our API, the "uid" must match customer ID on Stripe, Braintree, Chargebee, Recurly. Since Stripe doesn't allow pre-defined customer id, you can also pass the "uid" value as "fp_uid" in customer metadata later, when you create the customer object. |
| first_name | no                         | you can add the full name here in case you don't have access to both first and last name separately                                                                                                                                                                                                                                                                                                           |
| last_name  | no                         |                                                                                                                                                                                                                                                                                                                                                                                                               |
| tid        | required if ref_id is null | visitor tracking id. It's set when the visitor tracking script tracks the referral visit on our system. The value is found inside "\_fprom_track" cookie. Grab that value from the cookie and pass it here to match the lead with the referral.                                                                                                                                                               |
| ref_id     | required if tid is null    | default referral id of the promoter. Use this only when you want to assign the lead to a specific promoter.                                                                                                                                                                                                                                                                                                   |
| ip         | no                         | IP of the visitor who generated the sign up. It's used for fraud analysis.                                                                                                                                                                                                                                                                                                                                    |

## Tracking sales and commissions

```shell
curl -X POST "https://firstpromoter.com/api/v1/track/sale"
  -d "wid=b850ac4wer56hy1b5ef41"
  -d "email=shelley@example.com"
  -d "uid=cbdemo_shelley"
  -d "event_id=9856044"
  -d "plan=monthly-starter"
  -d "amount=6000"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "id": 1708,
  "type": "sale",
  "amount_cents": 6000,
  "lead": {
    "id": 943,
    "state": "active",
    "first_name": "Shelley",
    "last_name": "",
    "email": "shelley@example.com",
    "uid": "cbdemo_shelley",
    "customer_since": "2018-04-11T14:54:32.014Z",
    "plan_name": "monthly-starter",
    "suspicion": "no_suspicion"
  },
  "promoter": {
    "id": 1983,
    "cust_id": null,
    "email": "test@test.com",
    "temp_password": "u1PptB",
    "default_promotion_id": 1986,
    "default_ref_id": "test_ref_id",
    "earnings_balance": {
      "cash": 1200
    },
    "current_balance": {
      "cash": 1200
    },
    "paid_balance": null
  }
}
```

To track sales and generate commissions correctly, you need to use this API call each time a non-zero amount sale is processed in your system, even if it comes from a recurring charge or one-time charge.

To avoid fraudulent sales, we don't use JS conversions pixels to track sales, which are very unreliable and can be faked easily. Instead, we use server-side tracking for all sales to make sure that a sale is tracked only when you actually get money in your billing account. To keep the same standards with the API, we recommend to make the sale API call(this call) only when you receive the confirmation of the sale from your billing provider, like from a webhook, an IPN or a success response from an API charge call.

You just need to pass the sale amount(before taxes) and we'll take care of the rest. The commissions/rewards will be calculated based on that amount and the plan id, in case you use the plan-level rewards feature.

Using email or uid parameters we identify the lead/customer who generated the sale which also helps us determine the promoter who owns the reward/commission. The lead is added inside our system either by the client signup tracking script when the user signs up or by calling the signup API endpoint.

**If we don't find the lead on our system, it means that the sale is not a referral sale and you'll get a 204 response. You don't have to identify yourself which sale is from referrals and which is not, we'll take care of that.**

<aside class="notice">
Note: <strong>amount</strong> and <strong>mrr</strong> parameters takes the value in cents, so if you track the amounts as float values in your system you need to <strong>multiply by 100</strong> before sending the request.
</aside>

### HTTP Request

`POST https://firstpromoter.com/api/v1/track/sale`

### URL Parameters

| Parameter  | Required | Description                                                                                                                                                                                                                   |
| ---------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| email      | yes      | email of the lead/sign-up                                                                                                                                                                                                     |
| event_id   | yes      | transaction or charge ID. It's required to avoid generating duplicate commissions/rewards in case you mistakenly send the same API call multiple times.                                                                       |
| amount     | yes      | the sale amount in cents before taxes and with discounts applied to it. It's used to calculate the commissions/rewards.                                                                                                       |
| uid        | no       | uid of the lead added on signup tracking                                                                                                                                                                                      |
| first_name | no       | update lead first name                                                                                                                                                                                                        |
| last_name  | no       | update lead last name                                                                                                                                                                                                         |
| plan       | no       | customer plan ID from the billing provider. It's used to calculate rewards in case you use plan-level rewards feature.                                                                                                        |
| mrr        | no       | sets the Monthly Recurring Revenue generated by the customer. It's used only for calculating the MRR generated by the program, not for calculating the commissions.                                                           |
| promo_code | no       | for promo code/coupon code tracking. If you gave a unique coupon to a promoter and you added it on his promotion, you can pass it here and it will CREATE a new lead and a sale for that promoter(if doesn't exists already). |

## Tracking refunds and negative commissions

```shell
curl -X POST "https://firstpromoter.com/api/v1/track/refund"
  -d "wid=b850ac4wer56hy1b5ef41"
  -d "email=shelley@example.com"
  -d "uid=cbdemo_shelley"
  -d "event_id=4567044"
  -d "sale_event_id=9856044"
  -d "amount=6000"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "id": 1709,
  "type": "refund",
  "amount_cents": -6000,
  "lead": {
    "id": 943,
    "state": "active",
    "first_name": "Shelley",
    "last_name": "",
    "email": "shelley@example.com",
    "uid": "cbdemo_shelley",
    "customer_since": "2018-04-11T14:54:32.014Z",
    "plan_name": "monthly-starter",
    "suspicion": "no_suspicion"
  },
  "promoter": {
    "id": 1983,
    "cust_id": null,
    "email": "test@test.com",
    "temp_password": "u1PptB",
    "default_promotion_id": 1986,
    "default_ref_id": "test_ref_id",
    "earnings_balance": null,
    "current_balance": null,
    "paid_balance": null
  }
}
```

Refund call is similar with the sale call. It works the same way, just that it will produce negative commissions.

### HTTP Request

`POST https://firstpromoter.com/api/v1/track/refund`

### URL Parameters

| Parameter     | Required | Description                                                                                                                                                                                                                                                                                                 |
| ------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| email         | yes      | email of the lead/sign-up                                                                                                                                                                                                                                                                                   |
| event_id      | yes      | transaction or refund event ID. It's required to avoid generating duplicate refunds in case you mistakenly send the same API call multiple times.                                                                                                                                                           |
| amount        | yes      | the refund amount in cents. It's used to calculate the negative commissions/rewards.                                                                                                                                                                                                                        |
| uid           | no       | uid of the lead added on signup tracking                                                                                                                                                                                                                                                                    |
| sale_event_id | no       | the event id of the sale for which the refund is processed. This value must match the event_id value sent in the sale tracking API call. (Note: This field is marked as optional, but if you track multiple products or change the commissions level often, it becomes required to track refunds correctly) |

## Tracking cancellations

```shell
curl -X POST "https://firstpromoter.com/api/v1/track/cancellation"
  -d "wid=b850ac4wer56hy1b5ef41"
  -d "email=shelley@example.com"
  -d "uid=cbdemo_shelley"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "id": 1710,
  "type": "cancellation",
  "amount_cents": null,
  "lead": {
    "id": 943,
    "state": "cancelled",
    "first_name": "Shelley",
    "last_name": "",
    "email": "shelley@example.com",
    "uid": "cbdemo_shelley",
    "customer_since": "2018-04-11T14:54:32.014Z",
    "plan_name": "monthly-starter",
    "suspicion": "no_suspicion"
  },
  "promoter": {
    "id": 1983,
    "cust_id": null,
    "email": "test@test.com",
    "temp_password": "u1PptB",
    "default_promotion_id": 1986,
    "default_ref_id": "test_ref_id",
    "earnings_balance": null,
    "current_balance": null,
    "paid_balance": null
  }
}
```

This call will mark the customer as cancelled and will decrease the MRR generated by him/her.

### HTTP Request

`POST https://firstpromoter.com/api/v1/track/cancellation`

### URL Parameters

| Parameter | Required | Description                              |
| --------- | -------- | ---------------------------------------- |
| email     | yes      | email of the lead/sign-up                |
| uid       | no       | uid of the lead added on signup tracking |
