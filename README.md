# Druid Director

## 1. Overlord Console
Druid Indexing Service 관련 모니터링
Apache-Druid 의 indexer console 기능.
Druid의 API 이용.

### 1.1 Supervisors
supervisor
  
|Field|Description|Example|  
|--------|-----------|-------|  
|`dataSource`| | |  
|`status`|running or suspend | | 
|`more`|`paylord` `status` `history` `suspend` `reset` `terminate`| |

### 1.2 Running Tasks
실행중인 tasks

|Field|Description|Example|  
|--------|-----------|-------|  
|`id`|taskId | index_xxxx_2018-11-09Txxxxx |  
|`type`| |index or index_kafka or index_hadoop | 
|`createdTime`| |2018-11-28T01:06:51.407Z |
|`queueInsertionTime`|? | |
|`statusCode`|? | |
|`status`|RUNNING or WAITING or SUCCESS or FAILED | |
|`runnerStatusCode`|runner의 statusCode. RUNNING or WAITING or NONE| |
|`duration`| task 실행 시간| |
|`locationhost`| task가 실행되는 host| |
|`locationport`| task가 실행되는 port| |
|`dataSource`| | |
|`more`|`paylord` `status` `history` `suspend` `reset` `terminate`| |

note: runningTasks, waitingTasks, completedTasks의 task 정보는 모두 동일

### 1.3 Pending Tasks
Tasks waiting to be assigned to a worker

여유있는 worker가 생기면, worker가 할당 될 예정인 tasks

### 1.4 Waiting Tasks
Tasks waiting on locks
lock을 기다리는 tasks. 
실제로는 runner에게 정보가 없어서 lock 을 받을 가능성이 거의 없고, restart 해도 waiting으로 갈 가능성이 높은 tasks들 . 죽이는 게 답.

### 1.5 Completed Tasks
Tasks recently completed

### 1.6 Remote Workers

|Field|Description|Example|  
|--------|-----------|-------|  
|`worker scheme`| | |  
|`worekr host`| | | 
|`worker ip`| | |
|`worker capacity`| | |
|`worker version`| | |
|`currCapacityUsed`| | |
|`availabilityGroups`|? | |
|`runningTasks`| | |
|`lastCompletedTaskTime`| | |
|`blacklistedUntil`| | |

## 2. Datasource console
Druid의 coordinator console 에서 cluster, datasource 부분.
Druid API 이용.

### 2.1 cluster
- segment 수
- 사용량 / 저장 가능 용량 (1.12 TB / 900TB)
-  historical node list
	- 각 historical 사용량 
### 2.2 datasource
- datasource list
	- 각 datasource 마다
	- 상태 표시 (파랑, 빨강..무슨 기준인지 ?)
	- interval 별 segment 사이즈, 정보 표시
	

## 3. Ingestion Metrics
Druid의 metrics을 이용.
Supervisor(index_kafka) 만 가능. --> overlord console의 supervisor 와 연결?

|Metric|Description|Dimensions|Normal Value|정보| 알람 | 기타 |  
|------|-----------|----------|------------|----------|----------|----------|  
|`ingest/events/thrownAway`|Number of events rejected because they are outside the windowPeriod.|dataSource, taskId, taskType.|0| OK | X |  | 
|`ingest/events/unparseable`|Number of events rejected because the events are unparseable.|dataSource, taskId, taskType.|0| OK | X |  |  
|`ingest/events/duplicate`|Number of events rejected because the events are duplicated.|dataSource, taskId, taskType.|0| OK | X |  |  
|`ingest/events/processed`|Number of events successfully processed per emission period.|dataSource, taskId, taskType.|Equal to your # of events per emission period. | OK | X | |  
|`ingest/rows/output`|Number of Druid rows persisted.|dataSource, taskId, taskType.|Your # of events with rollup.| OK | X |  |  
|`ingest/persists/count`|Number of times persist occurred.|dataSource, taskId, taskType.|Depends on configuration.| OK | X |  |  
|`ingest/persists/time`|Milliseconds spent doing intermediate persist.|dataSource, taskId, taskType.|Depends on configuration. Generally a few minutes at most.| OK | X |  |  
|`ingest/persists/cpu`|Cpu time in Nanoseconds spent on doing intermediate persist.|dataSource, taskId, taskType.|Depends on configuration. Generally a few minutes at most.|   | O |  | 
|`ingest/persists/backPressure`|Milliseconds spent creating persist tasks and blocking waiting for them to finish.|dataSource, taskId, taskType.|0 or very low|  | O |  | 
|`ingest/persists/failed`|Number of persists that failed.|dataSource, taskId, taskType.|0|  | O |  | 
|`ingest/handoff/failed`|Number of handoffs that failed.|dataSource, taskId, taskType.|0|  | O |  |  
|`ingest/merge/time`|Milliseconds spent merging intermediate segments|dataSource, taskId, taskType.|Depends on configuration. Generally a few minutes at most.|  | ? |  |  
|`ingest/merge/cpu`|Cpu time in Nanoseconds spent on merging intermediate segments.|dataSource, taskId, taskType.|Depends on configuration. Generally a few minutes at most.|  | ? |  |  
|`ingest/handoff/count`|Number of handoffs that happened.|dataSource, taskId, taskType.|Varies. Generally greater than 0 once every segment granular period if cluster operating normally|  | X |  |  
|`ingest/sink/count`|Number of sinks not handoffed.|dataSource, taskId, taskType.|1~3|  | X |  |  
|`ingest/events/messageGap`|Time gap between the data time in event and current system time.|dataSource, taskId, taskType.|Greater than 0, depends on the time carried in event |  | X |  |  
|`ingest/kafka/lag`|Applicable for Kafka Indexing Service. Total lag between the offsets consumed by the Kafka indexing tasks and latest offsets in Kafka brokers across all partitions. Minimum emission period for this metric is a minute.|dataSource.|Greater than 0, should not be a very high number |  | X | API 사용예정 |

## 4. Coordination metrics
coordinator의  segment 관련 metrics.
API와 겹치는 내용이 있는 지 확인 필요. (현재의 status라면 API로도 있을 듯)

|Metric|Description|Dimensions|Normal Value|정보| 알람 | 기타 |    
|------|-----------|----------|------------|----------|----------|----------|  
|`segment/assigned/count`|Number of segments assigned to be loaded in the cluster.|tier.|Varies.|O |X | |  
|`segment/moved/count`|Number of segments moved in the cluster.|tier.|Varies.| | | |  
|`segment/dropped/count`|Number of segments dropped due to being overshadowed.|tier.|Varies.|O |X | |  
|`segment/deleted/count`|Number of segments dropped due to rules.|tier.|Varies.|O |X | |rule?|  
|`segment/unneeded/count`|Number of segments dropped due to being marked as unused.|tier.|Varies.|O |X | |  
|`segment/cost/raw`|Used in cost balancing. The raw cost of hosting segments.|tier.|Varies.| | | |  
|`segment/cost/normalization`|Used in cost balancing. The normalization of hosting segments.|tier.|Varies.| | | |  
|`segment/cost/normalized`|Used in cost balancing. The normalized cost of hosting segments.|tier.|Varies.| | | |  
|`segment/loadQueue/size`|Size in bytes of segments to load.|server.|Varies.| | | |  
|`segment/loadQueue/failed`|Number of segments that failed to load.|server.|0| | | |  
|`segment/loadQueue/count`|Number of segments to load.|server.|Varies.| | | |  
|`segment/dropQueue/count`|Number of segments to drop.|server.|Varies.| | | |  
|`segment/size`|Size in bytes of available segments.|dataSource.|Varies.| | |API?|  
|`segment/count`|Number of available segments.|dataSource.|< max| | |API?|  
|`segment/overShadowed/count`|Number of overShadowed segments.||Varies.|  
|`segment/unavailable/count`|Number of segments (not including replicas) left to load until segments that should be loaded in the cluster are available for queries.|datasource.|0| | O| |  
|`segment/underReplicated/count`|Number of segments (including replicas) left to load until segments that should be loaded in the cluster are available for queries.|tier, datasource.|0| | | |

## 5. Query Metrics
query metrics 는 broker, historical, real-time 에서 지원

### 5.1 Broker

|Metric|Description|Dimensions|Normal Value|정보| 알람 | 기타 |   
|------|-----------|----------|------------|----------|----------|----------|   
|`query/time`|Milliseconds taken to complete a query.|Common: dataSource, type, interval, hasFilters, duration, context, remoteAddress, id. Aggregation Queries: numMetrics, numComplexMetrics. GroupBy: numDimensions. TopN: threshold, dimension.|< 1s|O | | |  
|`query/bytes`|number of bytes returned in query response.|Common: dataSource, type, interval, hasFilters, duration, context, remoteAddress, id. Aggregation Queries: numMetrics, numComplexMetrics. GroupBy: numDimensions. TopN: threshold, dimension.| | | | |  
|`query/node/time`|Milliseconds taken to query individual historical/realtime nodes.|id, status, server.|< 1s| | |historical metrics에 있는 정보는? |  
|`query/node/bytes`|number of bytes returned from querying individual historical/realtime nodes.|id, status, server.| | | | |  
|`query/node/ttfb`|Time to first byte. Milliseconds elapsed until broker starts receiving the response from individual historical/realtime nodes.|id, status, server.|< 1s| | | |  
|`query/node/backpressure`|Milliseconds that the channel to this node has spent suspended due to backpressure.|id, status, server.| | | | |  
|`query/intervalChunk/time`|Only emitted if interval chunking is enabled. Milliseconds required to query an interval chunk. This metric is deprecated and will be removed in the future because interval chunking is deprecated. See [Query Context](../querying/query-context.html).|id, status, chunkInterval (if interval chunking is enabled).|< 1s| | | |  
|`query/count`|number of total queries|This metric is only available if the QueryCountStatsMonitor module is included.||O | | |  
|`query/success/count`|number of queries successfully processed|This metric is only available if the QueryCountStatsMonitor module is included.||O| | |  
|`query/failed/count`|number of failed queries|This metric is only available if the QueryCountStatsMonitor module is included.||O |O | |  
|`query/interrupted/count`|number of queries interrupted due to cancellation or timeout|This metric is only available if the QueryCountStatsMonitor module is included.||O |O | |  
  
### 5.2 Historical  
  
|Metric|Description|Dimensions|Normal Value|정보| 알람 | 기타 |   
|------|-----------|----------|------------|----------|----------|----------|   
|`query/time`|Milliseconds taken to complete a query.|Common: dataSource, type, interval, hasFilters, duration, context, remoteAddress, id. Aggregation Queries: numMetrics, numComplexMetrics. GroupBy: numDimensions. TopN: threshold, dimension.|< 1s| | |broker의 `query/node/time`? |  
|`query/segment/time`|Milliseconds taken to query individual segment. Includes time to page in the segment from disk.|id, status, segment.|several hundred milliseconds| |O | |  
|`query/wait/time`|Milliseconds spent waiting for a segment to be scanned.|id, segment.|< several hundred milliseconds| | O| |  
|`segment/scan/pending`|Number of segments in queue waiting to be scanned.||Close to 0| | |O |  
|`query/segmentAndCache/time`|Milliseconds taken to query individual segment or hit the cache (if it is enabled on the historical node).|id, segment.|several hundred milliseconds| |O | |  
|`query/cpu/time`|Microseconds of CPU time taken to complete a query|Common: dataSource, type, interval, hasFilters, duration, context, remoteAddress, id. Aggregation Queries: numMetrics, numComplexMetrics. GroupBy: numDimensions. TopN: threshold, dimension.|Varies| |O | |  
|`query/count`|number of total queries|This metric is only available if the QueryCountStatsMonitor module is included.|| | | |  
|`query/success/count`|number of queries successfully processed|This metric is only available if the QueryCountStatsMonitor module is included.|| | | |  
|`query/failed/count`|number of failed queries|This metric is only available if the QueryCountStatsMonitor module is included.|| | O| |  
|`query/interrupted/count`|number of queries interrupted due to cancellation or timeout|This metric is only available if the QueryCountStatsMonitor module is included.|| |O | |  
  
### 5.3 Real-time  
  
|Metric|Description|Dimensions|Normal Value|정보| 알람 | 기타 |   
|------|-----------|----------|------------|----------|----------|----------|   
|`query/time`|Milliseconds taken to complete a query.|Common: dataSource, type, interval, hasFilters, duration, context, remoteAddress, id. Aggregation Queries: numMetrics, numComplexMetrics. GroupBy: numDimensions. TopN: threshold, dimension.|< 1s| |O |O|  
|`query/wait/time`|Milliseconds spent waiting for a segment to be scanned.|id, segment.|several hundred milliseconds| | | |  
|`segment/scan/pending`|Number of segments in queue waiting to be scanned.||Close to 0| | | |  
|`query/count`|number of total queries|This metric is only available if the QueryCountStatsMonitor module is included.|| | | |  
|`query/success/count`|number of queries successfully processed|This metric is only available if the QueryCountStatsMonitor module is included.||  
|`query/failed/count`|number of failed queries|This metric is only available if the QueryCountStatsMonitor module is included.|| |O | |  
|`query/interrupted/count`|number of queries interrupted due to cancellation or timeout|This metric is only available if the QueryCountStatsMonitor module is included.|| |O | |

## 6. General Health
Druid metrics 이용

### 6.1 Historical  
 cluster 정보의 historical 에서 같이 표현?
 API?
 
|Metric|Description|Dimensions|Normal Value|정보| 알람 | 기타 |  
|------|-----------|----------|------------|----------|----------|----------|  
|`segment/max`|Maximum byte limit available for segments.||Varies.| | | |  
|`segment/used`|Bytes used for served segments.|dataSource, tier, priority.|< max| | | |  
|`segment/usedPercent`|Percentage of space used by served segments.|dataSource, tier, priority.|< 100%| | | |  
|`segment/count`|Number of served segments.|dataSource, tier, priority.|Varies.| |O | |  
|`segment/pendingDelete`|On-disk size in bytes of segments that are waiting to be cleared out|Varies.| |O | |  
  
### 6.2 JVM  
  
These metrics are only available if the JVMMonitor module is included.  
  
|Metric|Description|Dimensions|Normal Value|정보| 알람 | 기타 |  
|------|-----------|----------|------------|----------|----------|----------|  
|`jvm/pool/committed`|Committed pool.|poolKind, poolName.|close to max pool| | | |  
|`jvm/pool/init`|Initial pool.|poolKind, poolName.|Varies.| | | |  
|`jvm/pool/max`|Max pool.|poolKind, poolName.|Varies.| | | |  
|`jvm/pool/used`|Pool used.|poolKind, poolName.|< max pool| | | |  
|`jvm/bufferpool/count`|Bufferpool count.|bufferPoolName.|Varies.| | | |  
|`jvm/bufferpool/used`|Bufferpool used.|bufferPoolName.|close to capacity| | | |  
|`jvm/bufferpool/capacity`|Bufferpool capacity.|bufferPoolName.|Varies.| | | |  
|`jvm/mem/init`|Initial memory.|memKind.|Varies.| | | |  
|`jvm/mem/max`|Max memory.|memKind.|Varies.|O |O | |  
|`jvm/mem/used`|Used memory.|memKind.|< max memory|O |O |Peon 빼고 나머지는 정보용. |  
|`jvm/mem/committed`|Committed memory.|memKind.|close to max memory| | | |  
|`jvm/gc/count`|Garbage collection count.|gcName (cms/g1/parallel/etc.), gcGen (old/young)|Varies.| |O | |  
|`jvm/gc/cpu`|Cpu time in Nanoseconds spent on garbage collection.|gcName, gcGen|Sum of `jvm/gc/cpu` should be within 10-30% of sum of `jvm/cpu/total`, depending on the GC algorithm used (reported by [`JvmCpuMonitor`](../configuration/index.html#enabling-metrics)) | |O | |

## 7. Sys  
System Monitoring 은 Druid 모니터링에서 할 게 아니라, 전체적인 System 모니터링에 포함되어야 한다.
그래도, 만약 한다면, mem, storage, cpu 정도.   
  
These metrics are only available if the SysMonitor module is included.  
  
|Metric|Description|Dimensions|Normal Value|정보| 알람 | 기타 |  
|------|-----------|----------|------------| ----------|----------|----------| 
|`sys/swap/free`|Free swap.||Varies.| | | |  
|`sys/swap/max`|Max swap.||Varies.| | | |  
|`sys/swap/pageIn`|Paged in swap.||Varies.| | | |  
|`sys/swap/pageOut`|Paged out swap.||Varies.| | | |  
|`sys/disk/write/count`|Writes to disk.|fsDevName, fsDirName, fsTypeName, fsSysTypeName, fsOptions.|Varies.| | | |  
|`sys/disk/read/count`|Reads from disk.|fsDevName, fsDirName, fsTypeName, fsSysTypeName, fsOptions.|Varies.| | | |  
|`sys/disk/write/size`|Bytes written to disk. Can we used to determine how much paging is occuring with regards to segments.|fsDevName, fsDirName, fsTypeName, fsSysTypeName, fsOptions.|Varies.| | | |  
|`sys/disk/read/size`|Bytes read from disk. Can we used to determine how much paging is occuring with regards to segments.|fsDevName, fsDirName, fsTypeName, fsSysTypeName, fsOptions.|Varies.| | | |  
|`sys/net/write/size`|Bytes written to the network.|netName, netAddress, netHwaddr|Varies.| | | |  
|`sys/net/read/size`|Bytes read from the network.|netName, netAddress, netHwaddr|Varies.| | | |  
|`sys/fs/used`|Filesystem bytes used.|fsDevName, fsDirName, fsTypeName, fsSysTypeName, fsOptions.|< max| | | |  
|`sys/fs/max`|Filesystesm bytes max.|fsDevName, fsDirName, fsTypeName, fsSysTypeName, fsOptions.|Varies.| | | |  
|`sys/mem/used`|Memory used.||< max| |O | |  
|`sys/mem/max`|Memory max.||Varies.| |O | |  
|`sys/storage/used`|Disk space used.|fsDirName.|Varies.| |O | |  
|`sys/cpu`|CPU used.|cpuName, cpuTime.|Varies.| |O | |
