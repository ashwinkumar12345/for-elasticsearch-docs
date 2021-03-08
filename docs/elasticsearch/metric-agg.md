---
layout: default
title: Metric Aggregations
parent: Aggregations
grand_parent: Elasticsearch
nav_order: 1
has_children: false
---

# Metric Aggregations

Metric aggregations let you perform simple calculations such as finding the minimum, maximum, and average values of a field.

Metric aggregations are of two forms: single-value metrics and multi-value metrics.

## Single-value metrics

Single-value metrics return a single metric. For example, `sum`, `min`, `max`, `avg`, `value_count`, and `cardinality`.

### sum, min, max, avg

The `sum` metric finds the sum of the numeric values of a field.

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "sum_taxful_total_price": {
      "sum": {
        "field": "taxful_total_price"
      }
    }
  }
}
```

#### Sample Response

```json
...
  "aggregations" : {
    "sum_taxful_total_price" : {
      "value" : 350884.12890625
    }
  }
}
```

Similarly, you can use `min`, `max`, and `avg` metrics to find the minimum, maximum, and average values of a field, respectively.

### cardinality

The `cardinality` metric counts the number of unique or distinct values of a field.

For example, you can calculate how many unique products are present in your eCommerce store:

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "unique_products": {
      "cardinality": {
        "field": "products.product_id"
      }
    }
  }
}
```

#### Sample response

```json
...
  "aggregations" : {
    "unique_products" : {
      "value" : 7033
    }
  }
}
```

The cardinality count is approximate.
If you had tens of thousands of products in your store, an accurate cardinality calculation requires loading all values into a hash set and returning its size. This approach does not scale well because it requires more memory and causes high latency.

To trade memory for accuracy, you can set the `precision_threshold` setting. This setting defines the threshold below which counts are expected to be close to accurate. Above this value, counts might become a bit less accurate. The default value of `precision_threshold` is 3,000. The maximum supported value is 40,000.

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "unique_products": {
      "cardinality": {
        "field": "products.product_id",
        "precision_threshold": 10000
      }
    }
  }
}
```

### value_count

The `value_count` metric calculates the number of values that an aggregation is based on.
For example, you can use the `value_count` metric with the `avg` metric to see how many numbers the average value is based on.

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
   "aggs": {
    "number_of_values": {
      "value_count": {
        "field": "taxful_total_price"
      }
    }
  }
}
```

#### Sample response

```json
...
  "aggregations" : {
    "number_of_values" : {
      "value" : 4675
    }
  }
}
```

## Multi-value metrics

Multi-value metrics return more than one metric. For example, `stats` and `extended_stats`.

### stats

The `stats` aggregation returns all basic metrics such as `min`, `max`, `sum`, `avg`, and `value_count` metrics in one aggregation query.

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "stats_taxful_total_price": {
      "stats": {
        "field": "taxful_total_price"
      }
    }
  }
}
```

#### Sample response

```json
...
"aggregations" : {
  "stats_taxful_total_price" : {
    "count" : 4675,
    "min" : 6.98828125,
    "max" : 2250.0,
    "avg" : 75.05542864304813,
    "sum" : 350884.12890625
  }
 }
}
```

The `extended_stats` aggregation is an extended version of the `stats` aggregation. Apart from including basic stats, `extended_stats` also provides stats such as `sum_of_squares`, `variance`, and `std_deviation` on the values of a field.

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "extended_stats_taxful_total_price": {
      "extended_stats": {
        "field": "taxful_total_price"
      }
    }
  }
}
```

#### Sample Response

```json
...
"aggregations" : {
  "extended_stats_taxful_total_price" : {
    "count" : 4675,
    "min" : 6.98828125,
    "max" : 2250.0,
    "avg" : 75.05542864304813,
    "sum" : 350884.12890625,
    "sum_of_squares" : 3.9367749294174194E7,
    "variance" : 2787.59157113862,
    "variance_population" : 2787.59157113862,
    "variance_sampling" : 2788.187974983536,
    "std_deviation" : 52.79764740155209,
    "std_deviation_population" : 52.79764740155209,
    "std_deviation_sampling" : 52.80329511482722,
    "std_deviation_bounds" : {
      "upper" : 180.6507234461523,
      "lower" : -30.53986616005605,
      "upper_population" : 180.6507234461523,
      "lower_population" : -30.53986616005605,
      "upper_sampling" : 180.66201887270256,
      "lower_sampling" : -30.551161586606312
    }
  }
 }
}
```

The `std_deviation_bounds` object provides a visual variance of the data with an interval of plus/minus two standard deviations from the mean.
To set the standard deviation to different value, say 3, you can set `sigma` to 3:

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "extended_stats_taxful_total_price": {
      "extended_stats": {
        "field": "taxful_total_price",
        "sigma": 3
      }
    }
  }
}
```

### matrix_stats

The `matrix_stats` aggregation lets you generate advanced stats for multiple fields in a matrix form.

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "matrix_stats_taxful_total_price": {
      "matrix_stats": {
        "fields": ["taxful_total_price", "products.base_price"]
      }
    }
  }
}
```

#### Sample response

```json
...
"aggregations" : {
  "matrix_stats_taxful_total_price" : {
    "doc_count" : 4675,
    "fields" : [
      {
        "name" : "products.base_price",
        "count" : 4675,
        "mean" : 34.994239430147196,
        "variance" : 360.5035285833703,
        "skewness" : 5.530161335032702,
        "kurtosis" : 131.16306324042148,
        "covariance" : {
          "products.base_price" : 360.5035285833703,
          "taxful_total_price" : 846.6489362233166
        },
        "correlation" : {
          "products.base_price" : 1.0,
          "taxful_total_price" : 0.8444765264325268
        }
      },
      {
        "name" : "taxful_total_price",
        "count" : 4675,
        "mean" : 75.05542864304839,
        "variance" : 2788.1879749835402,
        "skewness" : 15.812149139924037,
        "kurtosis" : 619.1235507385902,
        "covariance" : {
          "products.base_price" : 846.6489362233166,
          "taxful_total_price" : 2788.1879749835402
        },
        "correlation" : {
          "products.base_price" : 0.8444765264325268,
          "taxful_total_price" : 1.0
        }
      }
    ]
  }
 }
}
```

Stats measure | Description
:--- | :---
`count` | The number of samples measured.
`mean` | The average value of the field measured from the sample.
`variance` | How far the values of the field measured are spread out from its mean value. The larger the variance, the more it's spread from its mean value.
`skewness` | An asymmetric measure of the distribution of the field's values around the mean.
`kurtosis` | A measure of the tail heaviness of a distribution. As the tail becomes lighter, kurtosis decreases. As the tail becomes heavier, kurtosis increases.
`covariance` | A measure of the joint variability between two fields. A positive value means their values move in the same direction and vice versa.
`correlation` | A measure of the strength of the relationship between two fields. The valid values are between [-1, 1]. A value of -1 means the value is negatively correlated and a value of 1 means it is positively correlated. A value of 0 means there is no identifiable relationship between them.

### percentile

Percentile is the percentage of the data that's at or below certain threshold values. You can use percentiles to find outliers in data or figure out the distribution of data. Like the cardinality aggregation, the percentile aggregation is approximate.

This example calculates the percentile in relation to the `taxful_total_price` field:

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "percentile_taxful_total_price": {
      "percentiles": {
        "field": "taxful_total_price"
      }
    }
  }
}
```

#### Sample response

```json
...
"aggregations" : {
  "percentile_taxful_total_price" : {
    "values" : {
      "1.0" : 21.984375,
      "5.0" : 27.984375,
      "25.0" : 44.96875,
      "50.0" : 64.22061688311689,
      "75.0" : 93.0,
      "95.0" : 156.0,
      "99.0" : 222.0
    }
  }
 }
}
```

### percentile_ranks

Percentile rank is the percentile of values at or below a threshold grouped by a specified value. For example, if a value is greater than or equal to 80% of the values, it has a percentile rank of 80.

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "percentile_rank_taxful_total_price": {
      "percentile_ranks": {
        "field": "taxful_total_price",
        "values": [
          10,
          15
        ]
      }
    }
  }
}
```

#### Sample response

```json
...
"aggregations" : {
  "percentile_rank_taxful_total_price" : {
    "values" : {
      "10.0" : 0.055096056411283456,
      "15.0" : 0.0830092961834656
    }
  }
 }
}
```

### geo_bound

The geo_bound metric calculates the bounding box in terms of latitude and longitude with respect to a `geo_point` field:

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "geo": {
      "geo_bounds": {
        "field": "geoip.location"
      }
    }
  }
}
```

#### Sample response

```json
"aggregations" : {
  "geo" : {
    "bounds" : {
      "top_left" : {
        "lat" : 52.49999997206032,
        "lon" : -118.20000001229346
      },
      "bottom_right" : {
        "lat" : 4.599999985657632,
        "lon" : 55.299999956041574
      }
    }
  }
 }
}
```

### top_hits

The `top_hits` metric ranks the matching documents based on a relevance score for the field that's being aggregated.

You can specify the following options:

- `from`: The starting position of the hit.
- `size`: The maximum size of hits to return. The default value is 3.
- `sort`: How the matching hits are sorted. By default, the hits are sorted by the relevance score of the aggregation query.

```json
GET kibana_sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "top_hits_taxful_total_price": {
      "top_hits": {
        "size": 5
      }
    }
  }
}
```

#### Sample response

```json
...
"aggregations" : {
  "top_hits_taxful_total_price" : {
    "hits" : {
      "total" : {
        "value" : 4675,
        "relation" : "eq"
      },
      "max_score" : 1.0,
      "hits" : [
        {
          "_index" : "kibana_sample_data_ecommerce",
          "_type" : "_doc",
          "_id" : "glMlwXcBQVLeQPrkHPtI",
          "_score" : 1.0,
          "_source" : {
            "category" : [
              "Women's Accessories",
              "Women's Clothing"
            ],
            "currency" : "EUR",
            "customer_first_name" : "rania",
            "customer_full_name" : "rania Evans",
            "customer_gender" : "FEMALE",
            "customer_id" : 24,
            "customer_last_name" : "Evans",
            "customer_phone" : "",
            "day_of_week" : "Sunday",
            "day_of_week_i" : 6,
            "email" : "rania@evans-family.zzz",
            "manufacturer" : [
              "Tigress Enterprises"
            ],
            "order_date" : "2021-02-28T14:16:48+00:00",
            "order_id" : 583581,
            "products" : [
              {
                "base_price" : 10.99,
                "discount_percentage" : 0,
                "quantity" : 1,
                "manufacturer" : "Tigress Enterprises",
                "tax_amount" : 0,
                "product_id" : 19024,
                "category" : "Women's Accessories",
                "sku" : "ZO0082400824",
                "taxless_price" : 10.99,
                "unit_discount_amount" : 0,
                "min_price" : 5.17,
                "_id" : "sold_product_583581_19024",
                "discount_amount" : 0,
                "created_on" : "2016-12-25T14:16:48+00:00",
                "product_name" : "Snood - white/grey/peach",
                "price" : 10.99,
                "taxful_price" : 10.99,
                "base_unit_price" : 10.99
              },
              {
                "base_price" : 32.99,
                "discount_percentage" : 0,
                "quantity" : 1,
                "manufacturer" : "Tigress Enterprises",
                "tax_amount" : 0,
                "product_id" : 19260,
                "category" : "Women's Clothing",
                "sku" : "ZO0071900719",
                "taxless_price" : 32.99,
                "unit_discount_amount" : 0,
                "min_price" : 17.15,
                "_id" : "sold_product_583581_19260",
                "discount_amount" : 0,
                "created_on" : "2016-12-25T14:16:48+00:00",
                "product_name" : "Cardigan - grey",
                "price" : 32.99,
                "taxful_price" : 32.99,
                "base_unit_price" : 32.99
              }
            ],
            "sku" : [
              "ZO0082400824",
              "ZO0071900719"
            ],
            "taxful_total_price" : 43.98,
            "taxless_total_price" : 43.98,
            "total_quantity" : 2,
            "total_unique_products" : 2,
            "type" : "order",
            "user" : "rani",
            "geoip" : {
              "country_iso_code" : "EG",
              "location" : {
                "lon" : 31.3,
                "lat" : 30.1
              },
              "region_name" : "Cairo Governorate",
              "continent_name" : "Africa",
              "city_name" : "Cairo"
            },
            "event" : {
              "dataset" : "sample_ecommerce"
            }
          }
          ...
        }
      ]
    }
  }
 }
}
```

### Scripted metric

This metric aggregation returns metrics calculated from a specified script.

A scripts has four stages: the init stage, the map stage, the combine stage, and the reduce stage.

* `init_script`: (OPTIONAL) Sets the initial state and executes before any collection of documents.
* `map_script`: Checks the value of the type field and executes the aggregation on the collected documents.
* `combine_script`: Aggregates the state returned from every shard. The aggregated value is returned to the coordinating node.
* `reduce_script`: Provides access to the variable states; this variable collects the results from combine_script on each shard into an array.

For example, to count the different HTTP response types in web log data:

```json
GET kibana_sample_data_logs/_search
{
  "size": 0,
  "aggregations": {
    "responses.counts": {
      "scripted_metric": {
        "init_script": "state.responses = ['error':0L,'success':0L,'other':0L]",
        "map_script": """
              def code = doc['response.keyword'].value;
                 if (code.startsWith('5') || code.startsWith('4')) {
                  state.responses.error += 1 ;
                  } else if(code.startsWith('2')) {
                   state.responses.success += 1;
                  } else {
                  state.responses.other += 1;
                }
             """,
        "combine_script": "state.responses",
        "reduce_script": """
            def counts = ['error': 0L, 'success': 0L, 'other': 0L];
                for (responses in states) {
                 counts.error += responses['error'];
                  counts.success += responses['success'];
                counts.other += responses['other'];
        }
        return counts;
        """
      }
    }
  }
}
```

#### Sample Response

```json
...
"aggregations" : {
  "responses.counts" : {
    "value" : {
      "other" : 0,
      "success" : 12832,
      "error" : 1242
    }
  }
 }
}
```