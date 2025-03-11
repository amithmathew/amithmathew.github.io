---
title: "[Part 1 of 2] Using GenAI to make customer segmentation more fun!"
date: "2024-04-04T15:57:24-04:00"
description: "Most marketers are familiar with customer segmentation. Let's do some customer segmentation with ML and then use Google Gemini to add some sizzle!"
draft: True
tags: ["ai","llm","marketing", "customer-segmentation",]
---

Most marketers are familiar with Customer Segmentation. Imagine you ran an online retailer. You have thousands of customers, about whom there are a few details you know, such as the country they're from, or the device they used to browse or purchase items from your store, or what their most commonly purchased item categories are etc. Imagine that you now want to send a promotion out, you now have a few choices.

### The Email Blast that usually goes to Spam

You could just send out an email blast to all your customers. 

_"30% off on all cosmetics!"_

As somebody who personally doesn't use cosmetics, I might be a little irritated. Send enough of these and I might mark your email as Spam, or unsubscribe (_oh no!_).  

{{< figure src="/images/spam-bin.png" width=500 >}}

### Rule-based Segmentation

You could use a rule-based approach to segmentation. 

_"I know this product appeals to customers who are between ages of 20 and 50 and live in the United States. That's who I'll send this promotion to!"_

This works, but this assumes that you know the exact audience that your promoted products are going to appeal to, and are able to build the right filters to target them. A little hard to scale, especially if you have many categories of items or a diverse enough customer base.  

{{< figure src="/images/rule-based-segmentation.png" width=500 >}}

### Old-Fashioned ML

You could use old-fashioned ML! [_k_-means](https://en.wikipedia.org/wiki/K-means_clustering) clustering to be precise. I think this makes WAY more sense than the other two approaches as it's scalable and easy to implement. 

{{< figure src="/images/old-fashioned-ml-superhero.png" width=500 >}}

Here's how to think about _k_-means. If I gave you a list of locations with two "features" - an x coordinate and a y coordinate and I tell you, 'Reader, I need to split these into two groups based on distance', how would you go about it? One approach would be to map these locations onto a 2-d plane, and "learn" where the hypothetical centers of two groups will lie on this plane given a particular way to calculate distance between points. In this case let's keep it trivial, "distance" is going to be actual distance between the two points on the plane.

Now once I've trained my model, given any new location, I can immediately calculate distance to the hypothetical group centers for the new point - the new location belongs to the group that it is closest to.

Now let's scale this up. Instead of a 2-d plane, we have a n-dimensional space - each dimension represents a particular "feature" of a customer - their lifetime total purchases, purchases in the last 30 days, ratio of activity to checkout etc. Now we can run the same _k_-means algorithm to group customers into different segments. The algorithm does expect you to tell it how many clusters to group the data into - so we'll look at ways to figure this out.

We'll Google Cloud BigQuery for a lot of the heavy lifting here, and streamlit to put a pretty face on it.

Let's get started!

## Getting Started

**Ingredients**
- 1 GCP project
- 1 user account with permissions to create datasets, tables and run SQL on BigQuery
- Access to BigQuery public datasets
- Access to Generative Models on VertexAI

We'll use fictitious data from [TheLook](https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce), one of the BigQuery's public datasets. The data model is straightforward and we'll use the following tables - 
1. `bigquery-public-data.thelook_ecommerce.products`  
2. `bigquery-public-data.thelook_ecommerce.orders`
3. `bigquery-public-data.thelook_ecommerce.order_items`
4. `bigquery-public-data.thelook_ecommerce.users`

## Mise-en-place
Let's build out our customer features. For each customer, I need to determine what dimensions I want to use to cluster them - these are our customer features.

I'm going to use the following because they were easy to compute, but there's better features to be found if you were to put some thought into it.

- Age
- Gender
- City
- Country
- Traffic Source
- Last 12 Months spend by Department-Category
- Last 24 Months spend by Department-Category
- Last 36 Months spend by Department-Category

I do this using the following SQL query.

```sql
create or replace table <my-project>.<my-dataset>.customer_features
as (
    with order_full as (
        select  o.order_id, o.user_id, o.status, o.gender, o.num_of_item, o.created_at,
                i.product_id, i.inventory_item_id, i.status as order_item_status, 
                i.sale_price, p.department, p.category  
        from    `bigquery-public-data.thelook_ecommerce.orders` o,
                `bigquery-public-data.thelook_ecommerce.order_items` i, 
                `bigquery-public-data.thelook_ecommerce.products` p
        where o.order_id = i.order_id
        and i.product_id = p.id
        order by created_at desc
    ),
    spend as (
        select  user_id,
                CONCAT(department, '_', category) as dep_cat,
                SUM(
                    CASE 
                        WHEN created_at 
                            BETWEEN TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL -365 DAY) 
                            AND CURRENT_TIMESTAMP()
                        THEN sale_price*num_of_item
                        ELSE 0
                    END
                ) AS total_12_months,
                SUM(
                    CASE 
                        WHEN created_at 
                            BETWEEN TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL -730 DAY) 
                            AND CURRENT_TIMESTAMP()
                        THEN sale_price*num_of_item
                        ELSE 0
                    END
                ) AS total_24_months,
                SUM(
                    CASE 
                        WHEN created_at 
                            BETWEEN TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL -1095 DAY) 
                            AND CURRENT_TIMESTAMP()
                        THEN sale_price*num_of_item
                        ELSE 0
                    END
                ) AS total_36_months
        from order_full
        group by user_id, dep_cat
    ),
    user_all as (
        select  s.user_id, 
                s.dep_cat,
                s.total_12_months,
                s.total_24_months,
                s.total_36_months,
                u.age,
                u.gender,
                u.city,
                u.country,
                u.traffic_source
        from    spend s, 
                `bigquery-public-data.thelook_ecommerce.users` u
        where s.user_id = u.id
    )
    select *
    from user_all
    PIVOT (
            SUM(total_12_months) as total_12_months,
            SUM(total_24_months) AS total_24_months,
            SUM(total_36_months) AS total_36_months
        FOR dep_cat IN (
            'Women_Accessories','Women_Plus','Men_Accessories',
            'Women_Swim','Women_Active','Women_Socks_Hosiery',
            'Men_Active','Men_Socks','Men_Swim',
            'Women_Dresses','Women_Pants_Capris','Men_FashionHoodies_Sweatshirts',
            'Women_Skirts','Women_Blazers_Jackets','Women_Suits',
            'Women_Tops_Tees','Women_Sweaters','Women_FashionHoodies_Sweatshirts',
            'Women_Shorts','Women_Jeans','Women_Maternity','Men_Shorts',
            'Men_Sleep_Lounge','Men_Suits_SportCoats','Men_Pants',
            'Women_Intimates','Women_Sleep_Lounge','Women_Outerwear_Coats',
            'Men_Tops_Tees','Men_Underwear','Men_Jeans','Men_Sweaters',
            'Women_Leggings','Women_Jumpsuits_Rompers','Men_Outerwear_Coats',
            'Women_ClothingSets'
        )
    )
);
```

Yes, this is ugly - no, don't judge me. I need to specify the department-category combinations in order to do the PIVOT. I get that by running the following query.

```sql
SELECT replace(
        regexp_replace(
            STRING_AGG(
                distinct CONCAT("'", department, '_', category, "'")
            ), r'\s', ''), 
        '&', '_')
FROM `bigquery-public-data.thelook_ecommerce.products`;
```

Again, this is a safe space, no judging.

You should now have a `customer_features` table in your dataset (you did replace `my-project` and `my-dataset` in the SQL above with your project name and dataset name didn't you?). And it should look something like this.

{{< figure src="/images/gcp-bq-customer-segmentation-genai-ss1.png" width=500 >}}

Now - we have some nulls in our department-category pivot columns because not every customer has purchases for every department-category combination. And the _k_-means algorithm we'll be using doesn't like nulls. So we'll make them 0s. I don't like having to write down all those column names by hand, so running the following SQL should generate the query you need to run. You could probably `EXECUTE IMMEDIATE` it directly.

```sql
select 
    CONCAT(
        "UPDATE `<my-project>.<my-dataset>.customer_features` SET ", 
        STRING_AGG(
            CONCAT( column_name, " = ", "COALESCE(", column_name, ", 0) ")), 
                    'WHERE true;')
from `<my-project>`.<my-dataset>.INFORMATION_SCHEMA.COLUMNS
where table_name = 'customer_features'
and column_name like 'total%';
```

Don't forget to use your own project and dataset names here. The generated SQL is large and ugly looking, so I'll spare you from looking at it, but go ahead and run that.


## Let's Cook Part 1: _k_-means

Now's the fun part. BigQuery allows you to create models using SQL, a feature called BigQuery ML. We'll use this to [create](https://cloud.google.com/bigquery/docs/reference/standard-sql/bigqueryml-syntax-create-kmeans) our _k_-means model.

```sql
CREATE MODEL
  `<my-project>.<my-dataset>.customer_segments_model`
OPTIONS
  ( MODEL_TYPE='KMEANS',
    NUM_CLUSTERS=HPARAM_RANGE(3, 5),
    KMEANS_INIT_METHOD='KMEANS++',
    NUM_TRIALS=10) 
AS
    SELECT *
    FROM `<my-project>.<my-dataset>.customer_features`;
```

Some things to note here, we could just decide that we want to split our customers into say 4 segments, and set `NUM_CLUSTERS` to 4. However, probably better to let hyperparameter tuning figure this out for us. By specifying `HPARAM_RANGE(3, 5)` we tell it to look at 3, 4 and 5 clusters and pick the most optimum number of clusters based on our data. You will need to specify `NUM_TRIALS` to enable hyperparameter tuning. 

Similarly, for the initialization of the algorithm, we've set `KMEANS_INIT_METHOD` to `KMEANS++`. You can find more details about _k_-means++ [here](https://en.wikipedia.org/wiki/K-means%2B%2B).

The default distance measure is Euclidean (think actual distance between two points). But you could change that to Cosine distance by setting `DISTANCE_TYPE`.

This is going to take some time to complete. But once it's done, you should see a model turn up within your dataset under the `Models` dropdown. Clicking on the model and then clicking on the "Details" tab should give you more information on the model, including the number of segments it decided to settle on.

{{< figure src="/images/gcp-bq-customer-segmentation-genai-ss2.png" width=500 >}}


Congratulations! We have now taken our transactional data, cleaned it up, put it into a form that the _k_-means model can consume, and trained up a _k_-means model including hyperparameter optimization to find the ideal number of clusters.

In Part 2, we'll use this model to segment our customer data. We'll then use Google's Gemini Large Language model to generate some "flavour" text for each segment - a nice heading and a description that we'll surface using Streamlit.

Until then!