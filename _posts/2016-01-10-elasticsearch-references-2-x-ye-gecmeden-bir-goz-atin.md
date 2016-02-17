---
layout: post
title: Elasticsearch References 2.x'ye Geçmeden Bir Göz Atın
categories:
- blog
summary: Elasticsearch 2.x ile birlikte neler gidiyor, neler değişiyor. 2.x sürümüne taşınmadan önce hızlıca göz atabileceğiniz bir döküman.
---

Elasticsearch 2.0 ile birlikte silinen ya da ismi değişen özellikleri bir 
dökümanda toplayayım dedim baya uzun bir liste çıktı. Genel bilgi olması 
açısından sırasıyla başlıklar ve kısa açıklamaları aşağıdaki gibi.

### Nodes shutdown
`_shutdown` API silindi. Bunun yerine Elasticsearch'ü işletim sisteminizde 
`service` olarak çalıştırabilirsiniz ya da `-p` komut satırı özelliğini 
kullanarak PID'i bir dosyaya yazdırabilirsiniz.

### Bulk UDP API
Bulk UDP servisi silindi. Bunun yerine [`Bulk API`](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/docs-bulk.html)'ı 
kullanabilirsiniz. 

### Mapping Silmek
Bir `type` için mapping'i silme özelliği artık olmayacak. Bunun yerine index'i 
silip yeni bir mapping ile oluşturabilirsiniz. Burada bir index'de zor durumda 
kalmadıkça birden fazla type barındırmamak gerekiyor gibi.

### Index Status
`_status` API [Indices Stats](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/indices-stats.html)
ve [Indices Recovery API](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/indices-recovery.html) 
ile değiştirildi.

### `_analyzer`
Type Mapping'deki `_analyzer` alanı artık desteklenmeyecek ve 2.x ile birlikte 
mapping'den otomatik olarak silinecek.

### `_boost`
Type Mapping'deki `_boost` alanı artık desteklenmeyecek ve 2.x ile birlikte 
mapping'den otomatik olarak silinecek.

### Config mappings
`config` klasöründe özel mapping'ler artık kullanılmayacak. Mapping oluşturma
artık aşağıdaki API arayüzleri ile olabilecek:

 - [Create Index](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/indices-create-index.html)
 - [Put Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/indices-put-mapping.html)
 - [Index Templates](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/indices-templates.html)

### memcached
`memcached` transport artık desteklenmeyecek. Bunun yerine HTTP veya Java API 
üzerinden REST arayüzünü kullanın.

### Thrift
`thrift` transport artık desteklenmeyecek. Bunun yerine HTTP veya Java API 
üzerinden REST arayüzünü kullanın.

### Queries, Filters
Query'ler ve Filter'lar birleştirildi. Herhangi bir `query` deyimi artık 
`query context` içerisinde `query` olarak, `filter context` içeriside `filter`
olarak kullanılabilecek. (Daha fazla bilgi için [`Query DSL`](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl.html))


| Silinen Filter'lar        | Kullanılabilecek Query Hali |
| ------------------------- | --------------------------- |
| And                       | [And Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-and-query.html)                             |
| Or                        | [Or Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-or-query.html)                               |
| Not                       | [Not Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-not-query.html)                             |
| Bool                      | [Bool Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-bool-query.html)                           |
| Exists                    | [Exists Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-exists-query.html)                       |
| Missing                   | [Missing Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-missing-query.html)                     |
| geo_bounding_box          | [Geo Bounding Box Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-geo-bounding-box-query.html)   |
| geo_distance              | [Geo Distance Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-geo-distance-query.html)           |
| geo_distance_range        | [Geo Distance Range Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-geo-distance-range-query.html) |
| geo_polygon               | [Geo Polygon Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-geo-polygon-query.html)             |
| geo_shape                 | [GeoShape Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-geo-shape-query.html)                  |
| geohash_cell              | [Geohash Cell Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-geohash-cell-query.html)           |
| has_child                 | [Has Child Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-has-child-query.html)                 |
| has_parent                | [Has Parent Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-has-parent-query.html)               |
| ids                       | [Ids Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-ids-query.html)                             |
| indices                   | [Indices Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-indices-query.html)                     |
| limit                     | [Limit Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-limit-query.html)                         |
| match_all                 | [Match All Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-match-all-query.html)                 |
| nested                    | [Nested Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-nested-query.html)                       |
| prefix                    | [Prefix Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-prefix-query.html)                       |
| query                     | -                         |
| range                     | [Range Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-range-query.html)                         |
| regexp                    | [Regexp Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-regexp-query.html)                       |
| script                    | [Script Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-script-query.html)                       |
| term                      | [Term Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-term-query.html)                           |
| terms                     | [Terms Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-terms-query.html)                         |
| type                      | [Type Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-type-query.html)                           |
| fuzzy_like_this veya flt  | [fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-match-query.html#query-dsl-match-query-fuzziness) parametresini `match` query ile kullanın veya [`More Like This Query`](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-mlt-query.html) kullanın. |
| fuzzy_like_this_field veya flt_field | [fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-match-query.html#query-dsl-match-query-fuzziness) parametresini `match` query ile kullanın veya [`More Like This Query`](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-mlt-query.html) kullanın. |

Bakınız: [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl.html)

### Top Children Query
`top_children` query silindi. Bunun yerine [`Has Child Query`](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-has-child-query.html) kullanın.

### More Like This API
`More Like This API` silindi. Bunun yerine [`More Like This Query`](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/query-dsl-mlt-query.html) kullanın.

### Facet'ler siliniyor `Aggregation`lar geliyor
`Facet`ler büyük veri kümelerinde özel bilgiler çıkarmak için çok güzel bir 
araçtır. Elasticsearch 1.0 ile birlikte `facet`lar `aggregation` olarak 
değişmiştir. `Aggregation`lar `facet`ların üst kümesidir. 

Aşağıda genel bir liste oluşturmaya çalıştım:

| Silinen `Facet`lar        | `Aggregation`lar          |
| ------------------------- | ------------------------- |
| Filter veya Query Facet   | [filter aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/search-aggregations-bucket-filter-aggregation.html) <br> [filters aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/search-aggregations-bucket-filters-aggregation.html) |
| Geo Distance Facet        | [geo_distance aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/search-aggregations-bucket-geodistance-aggregation.html) |
| Histogram Facet           | [histogram aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/search-aggregations-bucket-histogram-aggregation.html) |
| Date Histogram Facet      | [date_histogram aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/search-aggregations-bucket-datehistogram-aggregation.html) |
| Range Facet | [range aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/search-aggregations-bucket-range-aggregation.html) |
| Terms Facet | [terms aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/search-aggregations-bucket-terms-aggregation.html) |
| Terms Stats Facet | [terms aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/search-aggregations-bucket-terms-aggregation.html) <br> [stats aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/search-aggregations-metrics-stats-aggregation.html) <br> [extended_stats aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/search-aggregations-metrics-extendedstats-aggregation.html) |
| Statistical Facet | [stats aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/search-aggregations-metrics-stats-aggregation.html) <br> [extended_stats aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/2.x/search-aggregations-metrics-extendedstats-aggregation.html) |


### Shard request cache
`shard query cache`in ismi `Shard request cache` olarak değişti.

### Query cache
`filter cache`, `Node Query Cache` olarak değişti.

### `Nested tipi
The docs for the nested field datatype have moved to Nested datatype.


#### Kaynakça

 - [Breaking changes in 2.0](https://www.elastic.co/guide/en/elasticsearch/reference/current/breaking-changes-2.0.html)
 - [WHAT'S NEW IN ELASTICSEARCH 2.0](http://david.pilato.fr/presentations/what-s-new-in-elasticsearch-20.html)
