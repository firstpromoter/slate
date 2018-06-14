---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell



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

For increased security, in order to send a Tracking API call you'll need to pass the custom tracking integration ID(wid) as parameter with every call. Note: other API calls do not require this.

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

# Promoters API

Promoters API endpoint allows you to manage your affiliates/promoters through API calls. The most important use case is to automatically create promoter accounts for your customers.

To send an API call you will require the API key found in the "Settings" page to be added in the 'x-api-key' header.

## Create new promoters

```shell
curl -X POST "https://firstpromoter.com/api/v1/promoters/create"
  -d "email=john@doe.com"
  -d "first_name=John"
  -d "last_name=Doe"
  -d "cust_id=cus_sd4gh302fjlsd"
  -d "website=https://google.com"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "id": 2348,
  "cust_id": "cus_sd4gh302fjlsd",
  "email": "jon@doe.com",
  "temp_password": null,
  "default_promotion_id": 3341,
  "default_ref_id": "jon56",
  "earnings_balance": null,
  "current_balance": null,
  "paid_balance": null,
  "auth_token": "QvpsK_rzpzjYCBxfbATV8ubffmYDUf6u",
  "profile": {
    "id": 3390,
    "first_name": "John",
    "last_name": "Doe",
    "website": "https://google.com",
    "paypal_email": null,
    "avatar_url": null,
    "description": null,
    "social_accounts": {}
  }
}
```

Use this endpoint to create new promoters using the API. The response will return the newly added promoter as JSON.

Probably the most important value from the response is "auth_token", which is a unique key auto-generated on creation and can be used to automatically log in your promoters to their dashboard. [Learn more](https://help.firstpromoter.com/faq/how-to-embed-the-promoter-dashboard-inside-your-website-and-log-your-users-in-automatically)

<aside class="notice">
If this call is successful and you enabled "Promoter sign up accepted" emails for the campaign, it will start sending emails to the promoter. To skip the emails, set the <strong>skip_email_notification</strong> parameter to "true".
</aside>

### HTTP Request

`POST https://firstpromoter.com/api/v1/promoters/create`

### Query Parameters

| Parameter               | Required | Description                                                                                                                                                                                                                                                            |
| ----------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| email                   | yes      | promoter's email                                                                                                                                                                                                                                                       |
| first_name              | no       | promoter's first name                                                                                                                                                                                                                                                  |
| last_name               | no       | promoter's last name                                                                                                                                                                                                                                                   |
| cust_id                 | no       | customer/user ID inside your application. It will be used in webhooks to identify the promoter in your system.                                                                                                                                                         |
| ref_id                  | no       | referral ID. If this is blank an ID is assigned based on the first_name.                                                                                                                                                                                               |
| promo_code              | no       | unique promo code from your billing provider to assign to this affiliate for coupon tracking                                                                                                                                                                           |
| campaign_id             | no       | the ID of the campaign to assign the promoter to. On the campaigns sections you can see the id as "camp_id" query parameter on "Promoter Sign Up page URL". If there is no "camp_id" it means the campaign is the default campaign and this parameter is not required. |
| temp_password           | no       | a temporary password promoters can use to log in to their dashboard if you don't use authentication tokens(auth_token) to sign promoters in automatically.                                                                                                             |
| landing_url             | no       | you can set up a custom landing page url for this promoter. The referral id will be appended to it, unless the "url_tracking" parameter(below) is used.                                                                                                                |
| url_tracking            | no       | Set "true" to enable direct url tracking feature. FirstPromoter will do the tracking based on "landing_url"(above) without requiring the referral id to be appended to the url. The "landing_url" needs to be unique for each promoter. Default is "false".            |
| website                 | no       | promoter's website                                                                                                                                                                                                                                                     |
| paypal_email            | no       | promoter's Paypal Email address                                                                                                                                                                                                                                        |
| avatar_url              | no       | URL of the profile picture promoters can see on their dashboard                                                                                                                                                                                                        |
| description             | no       | A note/description to promoter                                                                                                                                                                                                                                         |
| skip_email_notification | no       | Set this to "true" to skip email notifications. Default is "false".                                                                                                                                                                                                    |

## Modify existing promoter

```shell
curl -X PUT "https://firstpromoter.com/api/v1/promoters/update"
  -d "cust_id=cus_sd4gh302fjlsd"
  -d "email=john_new@email.com"
  -d "new_ref_id=johnny"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "id": 2348,
  "cust_id": "cus_sd4gh302fjlsd",
  "email": "john_new@email.com",
  "temp_password": null,
  "default_promotion_id": 3341,
  "default_ref_id": "johnny",
  "earnings_balance": null,
  "current_balance": null,
  "paid_balance": null,
  "auth_token": "QvpsK_rzpzjYCBxfbATV8ubffmYDUf6u",
  "profile": {
    "id": 3390,
    "first_name": "John",
    "last_name": "Doe",
    "website": "https://google.com",
    "paypal_email": null,
    "avatar_url": null,
    "description": null,
    "social_accounts": {}
  }
}
```

You can identify promoters by: **id**, **cust_id**, **auth_token** or **ref_id**(referral id).
You need to pass at least one of these parameters.

### HTTP Request

`PUT https://firstpromoter.com/api/v1/promoters/update`

### Query Parameters

| Parameter     | Required | Description                                                                                                                                                                                                               |
| ------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id            | no       | promoter's ID inside FirstPromoter                                                                                                                                                                                        |
| cust_id       | no       | assigned customer/user ID                                                                                                                                                                                                 |
| ref_id        | no       | referral ID                                                                                                                                                                                                               |
| auth_token    | no       | authetication token generated when the promoter was created                                                                                                                                                               |
| new_cust_id   | no       | the new customer/user ID                                                                                                                                                                                                  |
| new_ref_id    | no       | the new referral ID                                                                                                                                                                                                       |
| email         | no       | promoter's email                                                                                                                                                                                                          |
| first_name    | no       | promoter's first name                                                                                                                                                                                                     |
| last_name     | no       | promoter's last name                                                                                                                                                                                                      |
| promo_code    | no       | unique promo code from your billing provider to assign to this affiliate for coupon tracking                                                                                                                              |
| temp_password | no       | a temporary password promoters can use to log in to their dashboard if you don't use authentication tokens(auth_token) to sign promoters in automatically.                                                                |
| landing_url   | no       | you can set up a custom landing page url for this promoter. The referral id will be appended to it, unless the "url_tracking" parameter(below) is used.                                                                   |
| url_tracking  | no       | enable direct url tracking feature. FirstPromoter will do the tracking based on "landing_url"(above) without requiring the referral id to be appended to the url. The "landing_url" needs to be unique for each promoter. |
| website       | no       | promoter's website                                                                                                                                                                                                        |
| paypal_email  | no       | promoter's Paypal Email address                                                                                                                                                                                           |
| avatar_url    | no       | URL of the profile picture promoters can see on their dashboard                                                                                                                                                           |
| description   | no       | A note/description to promoter                                                                                                                                                                                            |

## Show promoter details and balance

```shell
curl "https://firstpromoter.com/api/v1/promoters/show"
  -d "cust_id=cus_sd4gh302fjlsd"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "id": 2348,
  "cust_id": "cus_sd4gh302fjlsd",
  "email": "john_new@email.com",
  "temp_password": null,
  "default_promotion_id": 3341,
  "default_ref_id": "johnny",
  "earnings_balance": null,
  "current_balance": null,
  "paid_balance": null,
  "auth_token": "QvpsK_rzpzjYCBxfbATV8ubffmYDUf6u",
  "profile": {
    "id": 3390,
    "first_name": "John",
    "last_name": "Doe",
    "website": "https://google.com",
    "paypal_email": null,
    "avatar_url": null,
    "description": null,
    "social_accounts": {}
  }
}
```

You can identify promoters by: **id**, **cust_id**, **auth_token** or **ref_id**(referral id).
You need to pass at least one of these parameters.

### HTTP Request

`GET https://firstpromoter.com/api/v1/promoters/show`

### Query Parameters

| Parameter  | Required | Description                                                 |
| ---------- | -------- | ----------------------------------------------------------- |
| id         | no       | promoter's ID inside FirstPromoter                          |
| cust_id    | no       | assigned customer/user ID                                   |
| ref_id     | no       | referral ID                                                 |
| auth_token | no       | authetication token generated when the promoter was created |

## Reset promoter's authentication token

```shell
curl "https://firstpromoter.com/api/v1/promoters/show"
  -d "cust_id=cus_sd4gh302fjlsd"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "auth_token": "xXTqshq88jqnC3NBV56eALNnHMravuC9"
}
```

Resetting authentication token(auth_token) which is used to automatically sign in your promoters(if you use this feature) is not required, but can be used to enhance the security of the promoter dashboard. You can set up a cron job to call the API endpoint and store the new "auth_token" from the response.

You can identify promoters by: **id**, **cust_id** or **auth_token**...only one
required.

### HTTP Request

`PUT https://firstpromoter.com/api/v1/promoters/refresh_token`

### Query Parameters

| Parameter  | Required | Description                                                 |
| ---------- | -------- | ----------------------------------------------------------- |
| id         | no       | promoter's ID inside FirstPromoter                          |
| cust_id    | no       | assigned customer/user ID                                   |
| auth_token | no       | authetication token generated when the promoter was created |

## Delete a promoter

```shell
curl -X DELETE "https://firstpromoter.com/api/v1/promoters/delete"
  -d "cust_id=cus_sd4gh302fjlsd"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "message": "Promoter removed"
}
```

You can identify promoters by: **id**, **cust_id**, **auth_token** or **ref_id**(referral id).
You need to pass at least one of these parameters.

### HTTP Request

`DELETE https://firstpromoter.com/api/v1/promoters/delete`

### Query Parameters

| Parameter  | Required | Description                                                 |
| ---------- | -------- | ----------------------------------------------------------- |
| id         | no       | promoter's ID inside FirstPromoter                          |
| cust_id    | no       | assigned customer/user ID                                   |
| ref_id     | no       | referral ID                                                 |
| auth_token | no       | authetication token generated when the promoter was created |

# Leads API

Leads API allows you to manage the leads and customers referred by your promoters.

To send an API call you will require the API key found in the "Settings" page to be added in the 'x-api-key' header.

## Create new leads

```shell
curl -X POST "https://firstpromoter.com/api/v1/leads/create"
  -d "email=jane@doe.com"
  -d "ref_id=johnny"
  -d "uid=cus_r43d4lg9hhsd"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "id": 6925,
  "state": "signup",
  "email": "jane@doe.com",
  "uid": "cus_r43d4lg9hhsd",
  "customer_since": null,
  "plan_name": "",
  "suspicion": "no_suspicion",
  "promotion": {
    "id": 2736,
    "status": "offer_inactive",
    "ref_id": "johnny",
    "promo_code": null,
    "target_reached_at": null,
    "promoter_id": 2747,
    "campaign_id": 1286,
    "referral_link": "http://test.com#_r_johnny",
    "current_referral_reward": {
      "id": 205,
      "amount": 2000,
      "type": "per_referral",
      "unit": "cash",
      "name": "20% recurring commission",
      "default_promo_code": ""
    },
    "current_promotion_reward": null,
    "current_target_reward": null
  },
  "promoter": {
    "id": 2348,
    "cust_id": "cus_sd4gh302fjlsd",
    "email": "john_new@email.com",
    "temp_password": null,
    "default_promotion_id": 3341,
    "default_ref_id": "johnny",
    "earnings_balance": null,
    "current_balance": null,
    "paid_balance": null,
    "auth_token": "QvpsK_rzpzjYCBxfbATV8ubffmYDUf6u"
  }
}
```

With this endpoint you can assign a new lead/customer to a promoter using the API. You can find the promoter through its promotion either by **ref_id** or **promotion_id**.

### HTTP Request

`POST https://firstpromoter.com/api/v1/leads/create`

### Query Parameters

| Parameter      | Required                 | Description                                                                                     |
| -------------- | ------------------------ | ----------------------------------------------------------------------------------------------- |
| email          | yes                      | lead's email                                                                                    |
| promotion_id   | yes if ref_id null       | promotion id to assign the lead                                                                 |
| ref_id         | yes if promotion_id null | referral id of the promotion to assign the lead                                                 |
| uid            | no                       | id of the user on the blilling provider or in your database                                     |
| state          | no                       | lead's state. Can be **subscribed**,**signup**,**active** or **cancelled**                      |
| customer_since | no                       | time-date when lead converter to a customer                                                     |
| plan_name      | no                       | id of the plan the customer was assigned to. Needs to match with the plans set on FirstPromoter |

## Modify existing lead/customer

```shell
curl -X PUT "https://firstpromoter.com/api/v1/leads/update"
  -d "email=jane@doe.com"
  -d "new_uid=cus_updated"
  -d "plan_name=new_plan"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "id": 6925,
  "state": "signup",
  "email": "jane@doe.com",
  "uid": "cus_updated",
  "customer_since": null,
  "plan_name": "new_plan",
  "suspicion": "no_suspicion",
  "promotion": {
    "id": 2736,
    "status": "offer_inactive",
    "ref_id": "johnny",
    "promo_code": null,
    "target_reached_at": null,
    "promoter_id": 2747,
    "campaign_id": 1286,
    "referral_link": "http://test.com#_r_johnny",
    "current_referral_reward": {
      "id": 205,
      "amount": 2000,
      "type": "per_referral",
      "unit": "cash",
      "name": "20% recurring commission",
      "default_promo_code": ""
    },
    "current_promotion_reward": null,
    "current_target_reward": null
  },
  "promoter": {
    "id": 2348,
    "cust_id": "cus_sd4gh302fjlsd",
    "email": "john_new@email.com",
    "temp_password": null,
    "default_promotion_id": 3341,
    "default_ref_id": "johnny",
    "earnings_balance": null,
    "current_balance": null,
    "paid_balance": null,
    "auth_token": "QvpsK_rzpzjYCBxfbATV8ubffmYDUf6u"
  }
}
```

Use this to update a lead/customer details from FirstPromoter using the API. You can find the lead either by **id**, **uid** or **email**.

### HTTP Request

`PUT https://firstpromoter.com/api/v1/leads/update`

### Query Parameters

| Parameter      | Required                     | Description                                                                                     |
| -------------- | ---------------------------- | ----------------------------------------------------------------------------------------------- |
| id             | yes if email and uid is null | the lead id to update                                                                           |
| uid            | yes if id and email is null  | the lead uid to update                                                                          |
| email          | yes if id and uid is null    | the lead email to update                                                                        |
| new_uid        | no                           | the new uid                                                                                     |
| new_email      | no                           | the new email                                                                                   |
| state          | no                           | lead's state. Can be **subscribed**,**signup**,**active** or **cancelled**                      |
| customer_since | no                           | time-date when lead converter to a customer                                                     |
| plan_name      | no                           | id of the plan the customer was assigned to. Needs to match with the plans set on FirstPromoter |

## Show lead/customer details

```shell
curl "https://firstpromoter.com/api/v1/leads/show"
  -d "email=jane@doe.com"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "id": 6925,
  "state": "signup",
  "email": "jane@doe.com",
  "uid": "cus_updated",
  "customer_since": null,
  "plan_name": "new_plan",
  "suspicion": "no_suspicion",
  "promotion": {
    "id": 2736,
    "status": "offer_inactive",
    "ref_id": "johnny",
    "promo_code": null,
    "target_reached_at": null,
    "promoter_id": 2747,
    "campaign_id": 1286,
    "referral_link": "http://test.com#_r_johnny",
    "current_referral_reward": {
      "id": 205,
      "amount": 2000,
      "type": "per_referral",
      "unit": "cash",
      "name": "20% recurring commission",
      "default_promo_code": ""
    },
    "current_promotion_reward": null,
    "current_target_reward": null
  },
  "promoter": {
    "id": 2348,
    "cust_id": "cus_sd4gh302fjlsd",
    "email": "john_new@email.com",
    "temp_password": null,
    "default_promotion_id": 3341,
    "default_ref_id": "johnny",
    "earnings_balance": null,
    "current_balance": null,
    "paid_balance": null,
    "auth_token": "QvpsK_rzpzjYCBxfbATV8ubffmYDUf6u"
  }
}
```

Show the lead/customer details from FirstPromoter using the API. You can find the lead either by **id**, **uid** or **email**.

### HTTP Request

`GET https://firstpromoter.com/api/v1/leads/show`

### Query Parameters

| Parameter | Required                     | Description              |
| --------- | ---------------------------- | ------------------------ |
| id        | yes if email and uid is null | the lead id to update    |
| uid       | yes if id and email is null  | the lead uid to update   |
| email     | yes if id and uid is null    | the lead email to update |

## Delete a lead/customer

```shell
curl -X DELETE "https://firstpromoter.com/api/v1/leads/delete"
  -d "email=jane@doe.com"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "message": "Lead removed."
}
```

Remove a lead/customer from FirstPromoter using the API. You can find the lead either by **id**, **uid** or **email**.

### HTTP Request

`DELETE https://firstpromoter.com/api/v1/leads/delete`

### Query Parameters

| Parameter | Required                     | Description              |
| --------- | ---------------------------- | ------------------------ |
| id        | yes if email and uid is null | the lead id to update    |
| uid       | yes if id and email is null  | the lead uid to update   |
| email     | yes if id and uid is null    | the lead email to update |

# Rewards API

Rewards API allows you to create and assign custom rewards to promoters. It's
very helpful to create a custom referral program. For
example you can reward your customers for doing certain actions inside your
application.

To send an API call you will require the API key found in the "Settings" page to be added in the 'x-api-key' header.

## Create new reward

```shell
curl -X POST "https://firstpromoter.com/api/v1/rewards/create"
  -d "uid=cus_r43d4lg9hhsd"
  -d "reward_type=points"
  -d "amount=10"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "id": 6981,
  "status": "approved",
  "amount": 10,
  "unit": "points",
  "promoter": {
    "id": 2348,
    "cust_id": "cus_sd4gh302fjlsd",
    "email": "john_new@email.com",
    "temp_password": null,
    "default_promotion_id": 3341,
    "default_ref_id": "johnny",
    "earnings_balance": {
      "points": 10
    },
    "current_balance": {
      "points": 10
    },
    "paid_balance": null,
    "auth_token": "YhYtn86R3QhrYAMashi4yLMHnzEuSL2r"
  },
  "promotion": {
    "id": 2736,
    "status": "offer_inactive",
    "ref_id": "johnny",
    "promo_code": null,
    "target_reached_at": null,
    "promoter_id": 2747,
    "campaign_id": 1286,
    "referral_link": "http://test.com#_r_johnny",
    "current_referral_reward": {
      "id": 205,
      "amount": 2000,
      "type": "per_referral",
      "unit": "cash",
      "name": "20% recurring commission",
      "default_promo_code": ""
    },
    "current_promotion_reward": null,
    "current_target_reward": null
  }
}
```

With this endpoint you can assign a reward/commission generated by a lead/customer and it will be recorded to the promoter which referred the lead/customer. You can find the lead/customer by **lead_id**, **email** or **uid**.

If the reward/commission is not generated by a lead/customer, you can assign it
directly to a promoter through its promotion. You can find the promotion by
**promotion_id** or **ref_id**.

<aside class="notice">
Note: To assign rewards/commission for sales via the API use the <a
href="#tracking-api">Tracking API</a>
</aside>
<aside class="notice">
If this call is successful and you enabled "Promoter gets a new commission/reward" emails for the campaign, it will send emails to the promoter. To skip the emails, set the <strong>skip_email_notification</strong> parameter to "true".
</aside>

### HTTP Request

`POST https://firstpromoter.com/api/v1/rewards/create`

### Query Parameters

| Parameter               | Required | Description                                                                                                                                                       |
| ----------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| lead_id                 | no       | lead's id who generated the reward                                                                                                                                |
| email                   | no       | lead's email who generated the reward                                                                                                                             |
| uid                     | no       | lead's uid who generated the reward                                                                                                                               |
| promotion_id            | no       | promotion id of the promoter who owns the reward                                                                                                                  |
| ref_id                  | no       | promotion referral id of the who owns the reward                                                                                                                  |
| reward_type             | yes      | can be: **cash**(monetary commission), **points**, **credits**, **free_months**, **discount_per**(percentage discount), **discount_mon**(monetary fixed discount) |
| amount                  | yes      | amount of the reward. For reward_type **cash**(monetary commission) the amount is in cents                                                                        |
| status                  | no       | can be **approved**(default if this param is ommited), **pending** or **denied**                                                                                  |
| skip_email_notification | no       | Set this to "true" to skip email notifications. Default is "false".                                                                                               |
