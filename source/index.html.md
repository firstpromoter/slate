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

## Tracking leads and sign-ups

```shell
curl -X POST "https://firstpromoter.com/api/v1/track/signup"
  -d "email=shelley@example.com"
  -d "event_id=1001"
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

| Parameter | Required                   | Description                                                                                                                                                                                                                                                                                                                                                                                                   |
| --------- | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| event_id  | yes                        | signup event ID. It's required to avoid generating duplicate signup events in case you mistakenly send the same API call multiple times.                                                                                                                                                                                                                                                                      |
| email     | yes if uid is null         | email of the lead/sign-up                                                                                                                                                                                                                                                                                                                                                                                     |
| uid       | yes if email is null       | id to match the sale with the lead if the email can be changed before the first sale. If the sales are tracked by our built-in integrations and not by our API, the "uid" must match customer ID on Stripe, Braintree, Chargebee, Recurly. Since Stripe doesn't allow pre-defined customer id, you can also pass the "uid" value as "fp_uid" in customer metadata later, when you create the customer object. |
| tid       | required if ref_id is null | visitor tracking id. It's set when the visitor tracking script tracks the referral visit on our system. The value is found inside "\_fprom_track" cookie. Grab that value from the cookie and pass it here to match the lead with the referral.                                                                                                                                                               |
| ref_id    | required if tid is null    | default referral id of the promoter. Use this only when you want to assign the lead to a specific promoter.                                                                                                                                                                                                                                                                                                   |
| ip        | no                         | IP of the visitor who generated the sign up. It's used for fraud analysis.                                                                                                                                                                                                                                                                                                                                    |

## Tracking sales and commissions

```shell
curl -X POST "https://firstpromoter.com/api/v1/track/sale"
  -d "email=shelley@example.com"
  -d "uid=cbdemo_shelley"
  -d "currency=USD"
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

Using email or uid parameters we identify the lead/customer who generated the sale which also helps us determine the promoter who owns the reward/commission. The lead is added inside our system either by the client signup tracking script when the user signs up or by calling the signup API endpoint. There is also the option to bypass the signup tracking by using **tid** or **ref_id** parameters which will create the lead and also assign the sale in one go.

**If we don't find the lead on our system, it means that the sale is not a referral sale and you'll get a 204 response. You don't have to identify yourself which sale is from referrals and which is not, we'll take care of that.**

<aside class="notice">
Note: <strong>amount</strong> and <strong>mrr</strong> parameters takes the value in cents, so if you track the amounts as float values in your system you need to <strong>multiply by 100</strong> before sending the request.
</aside>

### HTTP Request

`POST https://firstpromoter.com/api/v1/track/sale`

### URL Parameters

| Parameter  | Required             | Description                                                                                                                                                                                                                         |
| ---------- | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| email      | yes if uid is null   | email of the lead/sign-up                                                                                                                                                                                                           |
| event_id   | yes                  | transaction or charge ID. It's required to avoid generating duplicate commissions/rewards in case you mistakenly send the same API call multiple times.                                                                             |
| amount     | yes                  | the total sale amount in cents before taxes and with discounts applied to it. It's used to calculate the commissions/rewards.                                                                                                       |
| uid        | yes if email is null | uid of the lead added on signup tracking                                                                                                                                                                                            |
| quantity   | no                   | number of subscriptions/items bought. If it's only one you can skip this parameter.                                                                                                                                                 |
| plan       | no                   | customer plan ID from the billing provider. It's used to calculate rewards in case you use plan-level rewards feature.                                                                                                              |
| currency   | no                   | this field is only required if the currency of the sale is not the same with the one set on FirstPromoter settings. We'll automatically convert the amount from this currency to the default one set on your FirstPromoter account. |
| mrr        | no                   | sets the Monthly Recurring Revenue generated by the customer. It's used only for calculating the MRR generated by the program, not for calculating the commissions.                                                                 |
| promo_code | no                   | for promo code/coupon code tracking. If you gave a unique coupon to a promoter and you added it on his promotion, you can pass it here and it will CREATE a new lead and a sale for that promoter(if doesn't exists already).       |
| tid        | no                   | you can avoid signup tracking call by providing the **\_fprom_track** cookie value(visitor tracking id) read on your system                                                                                                         |
| ref_id     | no                   | you can avoid signup tracking call by providing the **ref_id**(referral id) of the promoter                                                                                                                                         |

## Tracking refunds and negative commissions

```shell
curl -X POST "https://firstpromoter.com/api/v1/track/refund"
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

| Parameter     | Required             | Description                                                                                                                                                                                                                                                                                                 |
| ------------- | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| email         | yes if uid is null   | email of the lead/sign-up                                                                                                                                                                                                                                                                                   |
| event_id      | yes                  | transaction or refund event ID. It's required to avoid generating duplicate refunds in case you mistakenly send the same API call multiple times.                                                                                                                                                           |
| amount        | yes                  | the refund amount in cents. It's used to calculate the negative commissions/rewards.                                                                                                                                                                                                                        |
| quantity      | no                   | number of subscriptions/items refunded. If it's only one you can skip this parameter.                                                                                                                                                                                                                       |
| uid           | yes if email is null | uid of the lead added on signup tracking                                                                                                                                                                                                                                                                    |
| sale_event_id | no                   | the event id of the sale for which the refund is processed. This value must match the event_id value sent in the sale tracking API call. (Note: This field is marked as optional, but if you track multiple products or change the commissions level often, it becomes required to track refunds correctly) |

## Tracking cancellations

```shell
curl -X POST "https://firstpromoter.com/api/v1/track/cancellation"
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

| Parameter | Required             | Description                              |
| --------- | -------------------- | ---------------------------------------- |
| email     | yes if uid is null   | email of the lead/sign-up                |
| uid       | yes if email is null | uid of the lead added on signup tracking |

# Promoters API

Promoters API endpoint allows you to manage your affiliates/promoters through API calls. The most important use case is to automatically create promoter accounts for your customers.

To send an API call you will require the API key found in the "Settings" page to be added in the 'x-api-key' header.

## List promoters

```shell
curl -X GET "https://firstpromoter.com/api/v1/promoters/list"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
[
  {
    "id": 2347,
    "cust_id": "cus_s43t3fwef54",
    "email": "jane@doe.com",
    "temp_password": null,
    "default_promotion_id": 3340,
    "default_ref_id": "jane89",
    "earnings_balance": null,
    "current_balance": null,
    "paid_balance": null,
    "note": null,
    "auth_token": "as4fsQgrgEsdg4v5oiudv72FGSryyjs345",
    "profile": {
      "id": 3389,
      "first_name": "Jane",
      "last_name": "Doe",
      "website": "https://apple.com",
      "paypal_email": null,
      "avatar_url": null,
      "social_accounts": {}
    },
    "promotions": [
      {
        "id": 3340,
        "status": "offer_inactive",
        "ref_id": "jane89",
        "promo_code": null,
        "target_reached_at": null,
        "promoter_id": 2347,
        "campaign_id": 1286,
        "referral_link": "http://test.com?fp_ref=jane89",
        "current_referral_reward": {
          "id": 205,
          "amount": 2000,
          "type": "per_referral",
          "unit": "cash",
          "name": "20% recurring commission",
          "default_promo_code": ""
        },
        "current_promotion_reward": null,
        "current_target_reward": null,
        "visitors_count": 0,
        "leads_count": 0,
        "customers_count": 0,
        "refunds_count": 0,
        "cancellations_count": 0,
        "sales_count": 0,
        "sales_total": 0,
        "refunds_total": 0
      }
    ]
  },
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
    "note": null,
    "auth_token": "QvpsK_rzpzjYCBxfbATV8ubffmYDUf6u",
    "profile": {
      "id": 3390,
      "first_name": "John",
      "last_name": "Doe",
      "website": "https://google.com",
      "paypal_email": null,
      "avatar_url": null,
      "social_accounts": {}
    },
    "promotions": [
      {
        "id": 3341,
        "status": "offer_inactive",
        "ref_id": "jon56",
        "promo_code": null,
        "target_reached_at": null,
        "promoter_id": 2348,
        "campaign_id": 1286,
        "referral_link": "http://test.com?fp_ref=jon56",
        "current_referral_reward": {
          "id": 205,
          "amount": 2000,
          "type": "per_referral",
          "unit": "cash",
          "name": "20% recurring commission",
          "default_promo_code": ""
        },
        "current_promotion_reward": null,
        "current_target_reward": null,
        "visitors_count": 0,
        "leads_count": 0,
        "customers_count": 0,
        "refunds_count": 0,
        "cancellations_count": 0,
        "sales_count": 0,
        "sales_total": 0,
        "refunds_total": 0
      }
    ]
  }
]
```

Use this endpoint to list your promoters using the API. The response will return the promoters as JSON array.

<aside class="notice">
Pagination details are held on response headers. Add <strong>--include</strong> option on
<strong>curl</strong> request to see the pagination details format and links to next pages.
</aside>

### HTTP Request

`GET https://firstpromoter.com/api/v1/promoters/list`

### Query Parameters

| Parameter   | Required | Description                                         |
| ----------- | -------- | --------------------------------------------------- |
| campaign_id | no       | List all promoters accepted to a specific campaign. |

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
  "note": null,
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
  },
  "promotions": [
    {
      "id": 3341,
      "status": "offer_inactive",
      "ref_id": "jon56",
      "promo_code": null,
      "target_reached_at": null,
      "promoter_id": 2348,
      "campaign_id": 1286,
      "referral_link": "http://test.com#_r_jon56",
      "current_referral_reward": {
        "id": 205,
        "amount": 2000,
        "type": "per_referral",
        "unit": "cash",
        "name": "20% recurring commission",
        "default_promo_code": ""
      },
      "current_promotion_reward": null,
      "current_target_reward": null,
      "visitors_count": 0,
      "leads_count": 0,
      "customers_count": 0,
      "refunds_count": 0,
      "cancellations_count": 0,
      "sales_count": 0,
      "sales_total": 0,
      "refunds_total": 0
    }
  ]
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
| ref_id                  | no       | referral ID. If this is blank an ID is assigned based on the first\*name. Can be only lower-case letters, numbers, "-" and "\_"                                                                                                                                        |
| promo_code              | no       | unique promo code from your billing provider to assign to this affiliate for coupon tracking                                                                                                                                                                           |
| campaign_id             | no       | the ID of the campaign to assign the promoter to. On the campaigns sections you can see the id as "camp_id" query parameter on "Promoter Sign Up page URL". If there is no "camp_id" it means the campaign is the default campaign and this parameter is not required. |
| temp_password           | no       | a temporary password promoters can use to log in to their dashboard if you don't use authentication tokens(auth_token) to sign promoters in automatically.                                                                                                             |
| landing_url             | no       | you can set up a custom landing page url for this promoter. The referral id will be appended to it, unless the "url_tracking" parameter(below) is used.                                                                                                                |
| url_tracking            | no       | Set "true" to enable direct url tracking feature. FirstPromoter will do the tracking based on "landing_url"(above) without requiring the referral id to be appended to the url. The "landing_url" needs to be unique for each promoter. Default is "false".            |
| website                 | no       | promoter's website                                                                                                                                                                                                                                                     |
| paypal_email            | no       | promoter's Paypal Email address                                                                                                                                                                                                                                        |
| avatar_url              | no       | URL of the profile picture promoters can see on their dashboard                                                                                                                                                                                                        |
| note                    | no       | A note/description of promoter                                                                                                                                                                                                                                         |
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
  "note": null,
  "auth_token": "QvpsK_rzpzjYCBxfbATV8ubffmYDUf6u",
  "profile": {
    "id": 3390,
    "first_name": "John",
    "last_name": "Doe",
    "website": "https://google.com",
    "paypal_email": null,
    "avatar_url": null,
    "social_accounts": {}
  },
  "promotions": [
    {
      "id": 3341,
      "status": "offer_inactive",
      "ref_id": "johnny",
      "promo_code": null,
      "target_reached_at": null,
      "promoter_id": 2348,
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
      "current_target_reward": null,
      "visitors_count": 0,
      "leads_count": 0,
      "customers_count": 0,
      "refunds_count": 0,
      "cancellations_count": 0,
      "sales_count": 0,
      "sales_total": 0,
      "refunds_total": 0
    }
  ]
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
| new_ref_id    | no       | the new referral ID. Can be only lower-case letters, numbers, "-" and "\_"                                                                                                                                                |
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
| note          | no       | A note/description of promoter                                                                                                                                                                                            |

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
  "note": null,
  "auth_token": "QvpsK_rzpzjYCBxfbATV8ubffmYDUf6u",
  "profile": {
    "id": 3390,
    "first_name": "John",
    "last_name": "Doe",
    "website": "https://google.com",
    "paypal_email": null,
    "avatar_url": null,
    "social_accounts": {}
  },
  "promotions": [
    {
      "id": 3341,
      "status": "offer_inactive",
      "ref_id": "johnny",
      "promo_code": null,
      "target_reached_at": null,
      "promoter_id": 2348,
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
      "current_target_reward": null,
      "visitors_count": 0,
      "leads_count": 0,
      "customers_count": 0,
      "refunds_count": 0,
      "cancellations_count": 0,
      "sales_count": 0,
      "sales_total": 0,
      "refunds_total": 0
    }
  ]
}
```

You can identify promoters by: **id**, **cust_id**, **auth_token**, **promoter_email** or **ref_id**(referral id).
You need to pass at least one of these parameters.

### HTTP Request

`GET https://firstpromoter.com/api/v1/promoters/show`

### Query Parameters

| Parameter      | Required | Description                                                 |
| -------------- | -------- | ----------------------------------------------------------- |
| id             | no       | promoter's ID inside FirstPromoter                          |
| cust_id        | no       | assigned customer/user ID                                   |
| ref_id         | no       | referral ID                                                 |
| promoter_email | no       | promoter's email                                            |
| auth_token     | no       | authetication token generated when the promoter was created |

## Move/switch a promoter from one campaign to another

```shell
curl -X POST "https://firstpromoter.com/api/v1/promoters/move_to_campaign"
  -d "cust_id=cus_sd4gh302fjlsd"
  -d "destination_campaign_id=5399"
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
  "note": null,
  "auth_token": "QvpsK_rzpzjYCBxfbATV8ubffmYDUf6u",
  "profile": {
    "id": 3390,
    "first_name": "John",
    "last_name": "Doe",
    "website": "https://google.com",
    "paypal_email": null,
    "avatar_url": null,
    "social_accounts": {}
  },
  "promotions": [
    {
      "id": 3342,
      "status": "offer_inactive",
      "ref_id": "johnny",
      "promo_code": null,
      "target_reached_at": null,
      "promoter_id": 2348,
      "campaign_id": 5399,
      "referral_link": "http://test.com#_r_johnny",
      "current_referral_reward": {
        "id": 206,
        "per_of_sale": 30,
        "type": "per_referral",
        "unit": "cash",
        "name": "30% recurring commission",
        "default_promo_code": ""
      },
      "current_promotion_reward": null,
      "current_target_reward": null,
      "visitors_count": 0,
      "leads_count": 0,
      "customers_count": 0,
      "refunds_count": 0,
      "cancellations_count": 0,
      "sales_count": 0,
      "sales_total": 0,
      "refunds_total": 0
    }
  ]
}
```

This endpoint will change the campaign of the promoter with another one, keeping the referral id the same(affiliate
won't need to update the referral link). It's the same function to move the promoter to another campaign from the UI.
More details of how it works [here](https://help.firstpromoter.com/en/articles/2509650-moving-promoters-from-one-campaign-to-another).

You can identify promoters by: **id**, **cust_id**, **auth_token**, **promoter_email** or **ref_id**(referral id).

### HTTP Request

`POST https://firstpromoter.com/api/v1/promoters/move_to_campaign`

### Query Parameters

| Parameter               | Required | Description                                                                                                                                                                                  |
| ----------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id                      | no       | promoter's ID inside FirstPromoter                                                                                                                                                           |
| cust_id                 | no       | assigned customer/user ID                                                                                                                                                                    |
| ref_id                  | no       | referral ID                                                                                                                                                                                  |
| promoter_email          | no       | promoter's email                                                                                                                                                                             |
| auth_token              | no       | authetication token generated when the promoter was created                                                                                                                                  |
| destination_campaign_id | yes      | the id of the campaign to switch/move to. It can be found on the url bar when editing the campaign                                                                                           |
| source_campaign_id      | no       | only needed if the promoter is added to multiple campaigns. You can use this parameter to specify which campaign to change. If none is specified if will use as source the default campaign. |

## Add a promoter to a campaign

```shell
curl -X POST "https://firstpromoter.com/api/v1/promoters/add_to_campaign"
  -d "cust_id=cus_sd4gh302fjlsd"
  -d "campaign_id=5399"
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
  "note": null,
  "auth_token": "QvpsK_rzpzjYCBxfbATV8ubffmYDUf6u",
  "profile": {
    "id": 3390,
    "first_name": "John",
    "last_name": "Doe",
    "website": "https://google.com",
    "paypal_email": null,
    "avatar_url": null,
    "social_accounts": {}
  },
  "promotions": [
    {
      "id": 3341,
      "status": "offer_inactive",
      "ref_id": "johnny",
      "promo_code": null,
      "target_reached_at": null,
      "promoter_id": 2348,
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
      "current_target_reward": null,
      "visitors_count": 0,
      "leads_count": 0,
      "customers_count": 0,
      "refunds_count": 0,
      "cancellations_count": 0,
      "sales_count": 0,
      "sales_total": 0,
      "refunds_total": 0
    },
    {
      "id": 3342,
      "status": "offer_inactive",
      "ref_id": "johny23",
      "promo_code": null,
      "target_reached_at": null,
      "promoter_id": 2348,
      "campaign_id": 5399,
      "referral_link": "http://test.com#_r_johnny23",
      "current_referral_reward": {
        "id": 206,
        "per_of_sale": 30,
        "type": "per_referral",
        "unit": "cash",
        "name": "30% recurring commission",
        "default_promo_code": ""
      },
      "current_promotion_reward": null,
      "current_target_reward": null,
      "visitors_count": 0,
      "leads_count": 0,
      "customers_count": 0,
      "refunds_count": 0,
      "cancellations_count": 0,
      "sales_count": 0,
      "sales_total": 0,
      "refunds_total": 0
    }
  ]
}
```

Use this endpoint to add a promoter to another campaign. Your promoter will have multiple campaigns to promoter, each
campaign with its own promotion, referral link and referral id.

You can identify promoters by: **id**, **cust_id**, **auth_token**, **promoter_email** or **ref_id**(referral id).

### HTTP Request

`POST https://firstpromoter.com/api/v1/promoters/add_to_campaign`

### Query Parameters

| Parameter      | Required | Description                                                 |
| -------------- | -------- | ----------------------------------------------------------- |
| id             | no       | promoter's ID inside FirstPromoter                          |
| cust_id        | no       | assigned customer/user ID                                   |
| ref_id         | no       | referral ID                                                 |
| promoter_email | no       | promoter's email                                            |
| auth_token     | no       | authetication token generated when the promoter was created |
| campaign_id    | yes      | the id of the campaign you want to add to the promoter      |

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

## List leads and customers

```shell
curl -X GET "https://firstpromoter.com/api/v1/leads/list"
  -d "ref_id=johnny"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
[
  {
    "id": 6924,
    "state": "active",
    "email": "mike@doe.com",
    "uid": "cus_rxds34r9hhsd",
    "customer_since": "2018-11-11T14:54:32.081Z",
    "plan_name": "starter",
    "suspicion": "no_suspicion",
    "promotion": {
      "id": 3341,
      "status": "offer_inactive",
      "ref_id": "johnny",
      "promo_code": null,
      "target_reached_at": null,
      "promoter_id": 2348,
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
      "current_target_reward": null,
      "visitors_count": 0,
      "leads_count": 2,
      "customers_count": 1,
      "refunds_count": 0,
      "cancellations_count": 0,
      "sales_count": 0,
      "sales_total": 0,
      "refunds_total": 0
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
  },
  {
    "id": 6925,
    "state": "signup",
    "email": "jane@doe.com",
    "uid": "cus_r43d4lg9hhsd",
    "customer_since": null,
    "plan_name": "",
    "suspicion": "no_suspicion",
    "promotion": {
      "id": 3341,
      "status": "offer_inactive",
      "ref_id": "johnny",
      "promo_code": null,
      "target_reached_at": null,
      "promoter_id": 2348,
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
      "current_target_reward": null,
      "visitors_count": 0,
      "leads_count": 2,
      "customers_count": 1,
      "refunds_count": 0,
      "cancellations_count": 0,
      "sales_count": 0,
      "sales_total": 0,
      "refunds_total": 0
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
]
```

With this endpoint you can list all the leads and customers assigned to a promotion, promoter, campaign or entire account using the API.

<aside class="notice">
Pagination details are held on response headers. Add <strong>--include</strong> option on
<strong>curl</strong> request to see the pagination details format and links to next pages.
</aside>

### HTTP Request

`GET https://firstpromoter.com/api/v1/leads/list`

### Query Parameters

| Parameter    | Required | Description                                                                    |
| ------------ | -------- | ------------------------------------------------------------------------------ |
| promotion_id | no       | list all leads and customer assigned to a promotion                            |
| ref_id       | no       | list all leads and customer assigned to a promotion - find promotion by ref_id |
| promoter_id  | no       | list all leads and customers assigned to a promoter                            |
| campaign_id  | no       | list all leads and customers available to a campaign                           |

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
    "id": 3341,
    "status": "offer_inactive",
    "ref_id": "johnny",
    "promo_code": null,
    "target_reached_at": null,
    "promoter_id": 2348,
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
    "current_target_reward": null,
    "visitors_count": 0,
    "leads_count": 1,
    "customers_count": 0,
    "refunds_count": 0,
    "cancellations_count": 0,
    "sales_count": 0,
    "sales_total": 0,
    "refunds_total": 0
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

| Parameter      | Required                     | Description                                                                                                      |
| -------------- | ---------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| id             | yes if email and uid is null | the lead id to update                                                                                            |
| uid            | yes if id and email is null  | the lead uid to update                                                                                           |
| email          | yes if id and uid is null    | the lead email to update                                                                                         |
| new_uid        | no                           | the new uid                                                                                                      |
| new_email      | no                           | the new email                                                                                                    |
| new_ref_id     | no                           | if you want to move the lead or customer to another promoter, you can enter the referral id of the new promotion |
| state          | no                           | lead's state. Can be **subscribed**,**signup**,**active**,**denied** or **cancelled**                            |
| customer_since | no                           | time-date when lead converter to a customer                                                                      |
| plan_name      | no                           | id of the plan the customer was assigned to. Needs to match with the plans set on FirstPromoter                  |

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
    "id": 3341,
    "status": "offer_inactive",
    "ref_id": "johnny",
    "promo_code": null,
    "target_reached_at": null,
    "promoter_id": 2348,
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
    "current_target_reward": null,
    "visitors_count": 0,
    "leads_count": 1,
    "customers_count": 0,
    "refunds_count": 0,
    "cancellations_count": 0,
    "sales_count": 0,
    "sales_total": 0,
    "refunds_total": 0
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

Rewards API allows you to manage the rewards of your promoters. It's
very helpful to create a custom referral program. For example you can reward your customers for doing certain actions inside your
application.

To send an API call you will require the API key found in the "Settings" page to be added in the 'x-api-key' header.

## List rewards and commissions

```shell
curl -X GET "https://firstpromoter.com/api/v1/rewards/list"
  -d "ref_id=johnny"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
[
  {
    "id": 6981,
    "status": "approved",
    "amount": 10,
    "unit": "points",
    "conversion_ampunt": null,
    "event_id": null,
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
      "id": 3341,
      "status": "offer_inactive",
      "ref_id": "johnny",
      "promo_code": null,
      "target_reached_at": null,
      "promoter_id": 2348,
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
      "current_target_reward": null,
      "visitors_count": 0,
      "leads_count": 0,
      "customers_count": 0,
      "refunds_count": 0,
      "cancellations_count": 0,
      "sales_count": 0,
      "sales_total": 0,
      "refunds_total": 0
    }
  },
  {
    "id": 6982,
    "status": "denied",
    "amount": 10000,
    "event_id": "in_Xhf4Htyha3yGDd",
    "conversion_amount": 50000,
    "unit": "cash",
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
      "auth_token": "YhYtn86R3QhrYAMashi4yLMHnzEuSL2r"
    },
    "promotion": {
      "id": 3341,
      "status": "offer_inactive",
      "ref_id": "johnny",
      "promo_code": null,
      "target_reached_at": null,
      "promoter_id": 2348,
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
      "current_target_reward": null,
      "visitors_count": 0,
      "leads_count": 0,
      "customers_count": 0,
      "refunds_count": 0,
      "cancellations_count": 0,
      "sales_count": 0,
      "sales_total": 0,
      "refunds_total": 0
    }
  }
]
```

With this endpoint you can list all rewards and commissions assigned to a promotion, promoter, campaign or entire account using the API.

<aside class="notice">
Pagination details are held on response headers. Add <strong>--include</strong> option on
<strong>curl</strong> request to see the pagination details format and links to next pages.
</aside>

### HTTP Request

`GET https://firstpromoter.com/api/v1/rewards/list`

### Query Parameters

| Parameter    | Required | Description                                                                         |
| ------------ | -------- | ----------------------------------------------------------------------------------- |
| promotion_id | no       | list all rewards and commissions assigned to a promotion                            |
| ref_id       | no       | list all rewards and commissions assigned to a promotion - find promotion by ref_id |
| promoter_id  | no       | list all rewards and commissions assigned to a promoter                             |
| campaign_id  | no       | list all rewards and commissions available to a campaign                            |

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
  "conversion_ampunt": null,
  "event_id": null,
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
    "id": 3341,
    "status": "offer_inactive",
    "ref_id": "johnny",
    "promo_code": null,
    "target_reached_at": null,
    "promoter_id": 2348,
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
    "current_target_reward": null,
    "visitors_count": 0,
    "leads_count": 0,
    "customers_count": 0,
    "refunds_count": 0,
    "cancellations_count": 0,
    "sales_count": 0,
    "sales_total": 0,
    "refunds_total": 0
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

## Update a reward or commission

```shell
curl -X PUT "https://firstpromoter.com/api/v1/rewards/update"
  -d "event_id=in_Xhf4Htyha3yGDd"
  -d "status=denied"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
  "id": 6982,
  "status": "denied",
  "amount": 10000,
  "event_id": "in_Xhf4Htyha3yGDd",
  "conversion_amount": 50000,
  "unit": "cash",
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
    "auth_token": "YhYtn86R3QhrYAMashi4yLMHnzEuSL2r"
  },
  "promotion": {
    "id": 3341,
    "status": "offer_inactive",
    "ref_id": "johnny",
    "promo_code": null,
    "target_reached_at": null,
    "promoter_id": 2348,
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
    "current_target_reward": null,
    "visitors_count": 0,
    "leads_count": 0,
    "customers_count": 0,
    "refunds_count": 0,
    "cancellations_count": 0,
    "sales_count": 0,
    "sales_total": 0,
    "refunds_total": 0
  }
}
```

You can use this endpoint to change the status of a reward or commission. You can identify the reward or commission by
its id or the id of the event that generated the reward.

<aside class="notice">
For Stripe, Recurly and Chargebee subscription the event_id value is the Invoice id, for Stripe one time charges is the
charge id.
</aside>

### HTTP Request

`PUT https://firstpromoter.com/api/v1/rewards/update`

### Query Parameters

| Parameter | Required                | Description                                                              |
| --------- | ----------------------- | ------------------------------------------------------------------------ |
| id        | yes if event_id is null | id of the reward inside FirstPromoter                                    |
| event_id  | yes if id is null       | id of the event that generated the reward                                |
| status    | no                      | new status of the reward. Can be **approved**, **pending** or **denied** |

# Payouts API

Payouts API lets you to list and change the payouts status of your promoters.

To send an API call you will require the API key found in the "Settings" page to be added in the 'x-api-key' header.

## List payouts

```shell
curl -X GET "https://firstpromoter.com/api/v1/payouts/list"
  -d "campaign_id=1286"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
[
{
    "id": 78631,
    "status": "pending",
    "amount": 2000,
    "date_paid": null,
    "due_date": null,
    "unit": "cash",
    "created_at": "2020-12-01T00:16:13.390Z",
    "reward": null,
    "promoter": {
        "id": 2348,
        "cust_id": "cus_sd4gh302fjlsd",
        "email": "john_new@email.com",
        "temp_password": null,
        "default_promotion_id": 3341,
        "default_ref_id": "johnny",
        "auth_token": "YhYtn86R3QhrYAMashi4yLMHnzEuSL2r"
        "note": null,
        "earnings_balance": {
            "cash": 22000
        },
        "current_balance": {
            "cash": 22000
        },
        "pending_balance": {
            "cash": 2000
        },
        "paid_balance": null
    },
    "campaign": {
        "id": 1286,
        "name": "Affiliates",
        "landing_url": "http://test.com/",
        "description": "",
        "private": false,
        "color": "#00bcd4",
        "default_webhook_url": "",
        "auto_approve_rewards": true,
        "auto_approve_promoters": false
    }
},
{
    "id": 78632,
    "status": "completed",
    "amount": 5000,
    "date_paid": null,
    "due_date": null,
    "unit": "cash",
    "created_at": "2020-10-01T20:16:13.390Z",
    "reward": null,
    "promoter": {
        "id": 2350,
        "cust_id": "",
        "email": "amber@email.com",
        "temp_password": null,
        "default_promotion_id": 3371,
        "default_ref_id": "amber",
        "auth_token": "YhYtn86R34rfdk5j234adfsfa35EuSL2r"
        "note": null,
        "earnings_balance": {
            "cash": 5000
        },
        "current_balance": null,
        "paid_balance": {
            "cash": 5000
        },
    },
    "campaign": {
        "id": 1286,
        "name": "Affiliates",
        "landing_url": "http://test.com/",
        "description": "",
        "private": false,
        "color": "#00bcd4",
        "default_webhook_url": "",
        "auto_approve_rewards": true,
        "auto_approve_promoters": false
    }
}
]
```

With this endpoint you can list all payouts of a promoter, campaign or entire account using the API.

<aside class="notice">
Pagination details are held on response headers. Add <strong>--include</strong> option on
<strong>curl</strong> request to see the pagination details format and links to next pages.
</aside>

### HTTP Request

`GET https://firstpromoter.com/api/v1/payouts/list`

### Query Parameters

| Parameter   | Required | Description                                                                      |
| ----------- | -------- | -------------------------------------------------------------------------------- |
| promoter_id | no       | list all payouts assigned to a promoter                                          |
| campaign_id | no       | list all payouts of a campaign                                                   |
| status      | no       | filter payouts by status. Status can be **pending**,**processing**,**completed** |

## Change payouts status

```shell
curl -X PUT "https://firstpromoter.com/api/v1/payouts/update"
  -d "id=78632"
  -d "status=completed"
  -H "x-api-key: 2947d4543695e7cc7dhda3c52ebyt74eb8"
```

> Example response:

```json
{
    "id": 78632,
    "status": "completed",
    "amount": 5000,
    "date_paid": null,
    "due_date": null,
    "unit": "cash",
    "created_at": "2020-10-01T20:16:13.390Z",
    "reward": null,
    "promoter": {
        "id": 2350,
        "cust_id": "",
        "email": "amber@email.com",
        "temp_password": null,
        "default_promotion_id": 3371,
        "default_ref_id": "amber",
        "auth_token": "YhYtn86R34rfdk5j234adfsfa35EuSL2r"
        "note": null,
        "earnings_balance": {
            "cash": 5000
        },
        "current_balance": null,
        "paid_balance": {
            "cash": 5000
        },
    },
    "campaign": {
        "id": 1286,
        "name": "Affiliates",
        "landing_url": "http://test.com/",
        "description": "",
        "private": false,
        "color": "#00bcd4",
        "default_webhook_url": "",
        "auto_approve_rewards": true,
        "auto_approve_promoters": false
    }
}
```

This call allows you to change the status of the payout. For ex. you can mark the payout as completed once the payout
is paid.

### HTTP Request

`PUT https://firstpromoter.com/api/v1/payouts/update`

### Query Parameters

| Parameter | Required | Description                                                                   |
| --------- | -------- | ----------------------------------------------------------------------------- |
| id        | yes      | list all payouts assigned to a promoter                                       |
| status    | no       | the new payout status. Status can be **pending**,**processing**,**completed** |
