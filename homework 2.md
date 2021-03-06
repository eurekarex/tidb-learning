# 第二次作业

## TiDB集群设置
本次作业中，我把一个TiDB集群部署在Azure公有云中。我的TiDB集群包含1个TiDB，3个PD，3个TiKV，1个TiFlash。这些接点部署在5台Standard D4s v3 (4 vcpus, 16 GiB memory)虚拟机上，每台虚拟机配置一块30GB的Premium SSD硬盘。虚拟机安装Ubuntu 18.04 LTS版本操作系统。

其中TiDB，TiFlash分别独享一台虚拟机，一个PD和一个TiKV共享一个虚拟机。监测服务部署在TiDB所在的虚拟机。具体集群信息如下。

```
Starting component `cluster`:  display tidb-class
tidb Cluster: tidb-class
tidb Version: v4.0.4
ID              Role          Host      Ports                            OS/Arch       Status   Data Dir                 Deploy Dir
--              ----          ----      -----                            -------       ------   --------                 ----------
10.0.0.4:9093   alertmanager  10.0.0.4  9093/9094                        linux/x86_64  Up       data                     deploy/alertmanager-9093
10.0.0.4:3000   grafana       10.0.0.4  3000                             linux/x86_64  Up       -                        deploy/grafana-3000
10.0.0.5:2379   pd            10.0.0.5  2379/2380                        linux/x86_64  Up       data                     deploy/pd-2379
10.0.0.6:2379   pd            10.0.0.6  2379/2380                        linux/x86_64  Up       data                     deploy/pd-2379
10.0.0.7:2379   pd            10.0.0.7  2379/2380                        linux/x86_64  Up|L|UI  data                     deploy/pd-2379
10.0.0.4:9090   prometheus    10.0.0.4  9090                             linux/x86_64  Up       data                     deploy/prometheus-9090
10.0.0.4:4000   tidb          10.0.0.4  4000/10080                       linux/x86_64  Up       -                        deploy/tidb-4000
10.0.0.8:9000   tiflash       10.0.0.8  9000/8123/3930/20170/20292/8234  linux/x86_64  Up       /tidb-data/tiflash-9000  /tidb-deploy/tiflash-9000
10.0.0.5:20160  tikv          10.0.0.5  20160/20180                      linux/x86_64  Up       data                     deploy/tikv-20160
10.0.0.6:20160  tikv          10.0.0.6  20160/20180                      linux/x86_64  Up       data                     deploy/tikv-20160
10.0.0.7:20160  tikv          10.0.0.7  20160/20180                      linux/x86_64  Up       data                     deploy/tikv-20160
```
实例性能分析
![实例性能分析](resource/profiling_1_1_tidb_10_0_0_4_4000662501212.svg)

所有以下测试结果都是在该集群（tidb-class）上得出的数据。

## Sysbench
测试用的Sysbench版本1.0.20，使用版本自带的LuaJIT 2.1.0-beta2。
受限于集群的计算能力，大部分测试试用10000条数据，32张表作为测试数据，但为了测试不同数据库大小下集群的测试表现，我用100000条数剧，32张表的测试数据进行了point select的测试。
大部分测试使用32线程，但为了测试不同线程设置下集群的测试表现，我分别用16，128线程进行了个别测试。

### 数据准备
> sysbench --config-file=sysbench-config oltp_point_select --tables=32 --table-size=10000 --threads=32 prepare
### point select测试
> sysbench --config-file=sysbench-config oltp_point_select --tables=32 --table-size=10000 --threads=32 run
```
[ 10s ] thds: 32 tps: 23923.30 qps: 23923.30 (r/w/o: 23923.30/0.00/0.00) lat (ms,95%): 2.86 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 32 tps: 24917.48 qps: 24917.48 (r/w/o: 24917.48/0.00/0.00) lat (ms,95%): 2.66 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 32 tps: 23929.41 qps: 23929.41 (r/w/o: 23929.41/0.00/0.00) lat (ms,95%): 2.81 err/s: 0.00 reconn/s: 0.00
[ 40s ] thds: 32 tps: 23879.69 qps: 23879.69 (r/w/o: 23879.69/0.00/0.00) lat (ms,95%): 2.81 err/s: 0.00 reconn/s: 0.00
[ 50s ] thds: 32 tps: 24472.41 qps: 24472.41 (r/w/o: 24472.41/0.00/0.00) lat (ms,95%): 2.71 err/s: 0.00 reconn/s: 0.00
[ 60s ] thds: 32 tps: 24013.15 qps: 24013.15 (r/w/o: 24013.15/0.00/0.00) lat (ms,95%): 2.81 err/s: 0.00 reconn/s: 0.00
```
### update index测试
> sysbench --config-file=sysbench-config oltp_update_index --tables=32 --table-size=10000 --threads=32 run
```
[ 10s ] thds: 32 tps: 634.16 qps: 634.16 (r/w/o: 0.00/634.16/0.00) lat (ms,95%): 63.32 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 32 tps: 637.41 qps: 637.41 (r/w/o: 0.00/637.41/0.00) lat (ms,95%): 63.32 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 32 tps: 663.11 qps: 663.11 (r/w/o: 0.00/663.11/0.00) lat (ms,95%): 61.08 err/s: 0.00 reconn/s: 0.00
[ 40s ] thds: 32 tps: 598.78 qps: 598.78 (r/w/o: 0.00/598.78/0.00) lat (ms,95%): 59.99 err/s: 0.00 reconn/s: 0.00
[ 50s ] thds: 32 tps: 328.31 qps: 328.31 (r/w/o: 0.00/328.31/0.00) lat (ms,95%): 325.98 err/s: 0.00 reconn/s: 0.00
[ 60s ] thds: 32 tps: 680.82 qps: 680.82 (r/w/o: 0.00/680.82/0.00) lat (ms,95%): 58.92 err/s: 0.00 reconn/s: 0.00
```
### read only测试
> sysbench --config-file=sysbench-config oltp_read_only --tables=32 --table-size=10000 --threads=32 run
```
[ 10s ] thds: 32 tps: 550.37 qps: 8834.70 (r/w/o: 7730.86/0.00/1103.84) lat (ms,95%): 80.03 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 32 tps: 561.92 qps: 8989.39 (r/w/o: 7865.44/0.00/1123.95) lat (ms,95%): 78.60 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 32 tps: 548.91 qps: 8776.98 (r/w/o: 7679.47/0.00/1097.51) lat (ms,95%): 81.48 err/s: 0.00 reconn/s: 0.00
[ 40s ] thds: 32 tps: 547.19 qps: 8757.30 (r/w/o: 7663.21/0.00/1094.09) lat (ms,95%): 81.48 err/s: 0.00 reconn/s: 0.00
[ 50s ] thds: 32 tps: 551.50 qps: 8827.02 (r/w/o: 7723.72/0.00/1103.30) lat (ms,95%): 80.03 err/s: 0.00 reconn/s: 0.00
[ 60s ] thds: 32 tps: 536.80 qps: 8586.14 (r/w/o: 7512.73/0.00/1073.40) lat (ms,95%): 84.47 err/s: 0.00 reconn/s: 0.00
```
### 128线程下read only测试
> sysbench --config-file=sysbench-config oltp_read_only --tables=32 --table-size=10000 --threads=128 run
```
[ 10s ] thds: 128 tps: 561.36 qps: 9071.69 (r/w/o: 7937.78/0.00/1133.91) lat (ms,95%): 292.60 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 128 tps: 573.72 qps: 9184.37 (r/w/o: 8036.13/0.00/1148.23) lat (ms,95%): 287.38 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 128 tps: 583.11 qps: 9341.10 (r/w/o: 8174.89/0.00/1166.21) lat (ms,95%): 287.38 err/s: 0.00 reconn/s: 0.00
[ 40s ] thds: 128 tps: 579.91 qps: 9263.09 (r/w/o: 8103.47/0.00/1159.62) lat (ms,95%): 287.38 err/s: 0.00 reconn/s: 0.00
[ 50s ] thds: 128 tps: 587.70 qps: 9391.48 (r/w/o: 8217.08/0.00/1174.40) lat (ms,95%): 282.25 err/s: 0.00 reconn/s: 0.00
[ 60s ] thds: 128 tps: 586.09 qps: 9402.70 (r/w/o: 8229.13/0.00/1173.58) lat (ms,95%): 282.25 err/s: 0.00 reconn/s: 0.00
```

### 16线程下read only测试
> sysbench --config-file=sysbench-config oltp_read_only --tables=32 --table-size=10000 --threads=16 run
```
[ 10s ] thds: 16 tps: 517.49 qps: 8291.60 (r/w/o: 7255.12/0.00/1036.47) lat (ms,95%): 45.79 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 16 tps: 527.01 qps: 8433.38 (r/w/o: 7379.25/0.00/1054.12) lat (ms,95%): 44.98 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 16 tps: 522.60 qps: 8359.40 (r/w/o: 7314.40/0.00/1045.00) lat (ms,95%): 44.98 err/s: 0.00 reconn/s: 0.00
[ 40s ] thds: 16 tps: 521.11 qps: 8340.21 (r/w/o: 7297.90/0.00/1042.31) lat (ms,95%): 45.79 err/s: 0.00 reconn/s: 0.00
[ 50s ] thds: 16 tps: 534.09 qps: 8540.00 (r/w/o: 7471.91/0.00/1068.09) lat (ms,95%): 44.17 err/s: 0.00 reconn/s: 0.00
[ 60s ] thds: 16 tps: 528.49 qps: 8461.00 (r/w/o: 7404.11/0.00/1056.89) lat (ms,95%): 44.17 err/s: 0.00 reconn/s: 0.00
```
### 1000000条数据线point select测试
> sysbench --config-file=sysbench-config oltp_point_select --threads=32 --tables=32 --table-size=1000000 run
```
[ 10s ] thds: 32 tps: 16488.61 qps: 16488.61 (r/w/o: 16488.61/0.00/0.00) lat (ms,95%): 4.65 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 32 tps: 16739.07 qps: 16739.07 (r/w/o: 16739.07/0.00/0.00) lat (ms,95%): 4.49 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 32 tps: 17039.36 qps: 17039.36 (r/w/o: 17039.36/0.00/0.00) lat (ms,95%): 4.25 err/s: 0.00 reconn/s: 0.00
[ 40s ] thds: 32 tps: 16813.49 qps: 16813.49 (r/w/o: 16813.49/0.00/0.00) lat (ms,95%): 4.33 err/s: 0.00 reconn/s: 0.00
[ 50s ] thds: 32 tps: 16480.45 qps: 16480.45 (r/w/o: 16480.45/0.00/0.00) lat (ms,95%): 4.65 err/s: 0.00 reconn/s: 0.00
[ 60s ] thds: 32 tps: 16884.22 qps: 16884.32 (r/w/o: 16884.32/0.00/0.00) lat (ms,95%): 4.33 err/s: 0.00 reconn/s: 0.00
```

### 总结
![TiDB dashboard中各个节点的资源概况](resource/homework_2_sysbench_figure1.png)
![TiDB dashboard中流量可视化](resource/homework_2_sysbench_key_visulization.png)
[TiDB诊断报告](resource/dashboard_diagnose_report.pdf)
测试中主要显现出的瓶颈在TiDB节点。CPU资源是影响这次测试结果的主要原因。我认为原因主要是受限于我的测试集群的计算能力（所有虚拟机平均分配资源）。

## YCSB测试
我在编译go-ycsb的过程中遇到问题，尝试着修复编译的问题不过没有成功。于是改用ycsb进行以下测试。
### workloada
```
[OVERALL], RunTime(ms), 9707
[OVERALL], Throughput(ops/sec), 103.01844030081385
[TOTAL_GCS_G1_Young_Generation], Count, 1
[TOTAL_GC_TIME_G1_Young_Generation], Time(ms), 12
[TOTAL_GC_TIME_%_G1_Young_Generation], Time(%), 0.12362212836097661
[TOTAL_GCS_G1_Old_Generation], Count, 0
[TOTAL_GC_TIME_G1_Old_Generation], Time(ms), 0
[TOTAL_GC_TIME_%_G1_Old_Generation], Time(%), 0.0
[TOTAL_GCs], Count, 1
[TOTAL_GC_TIME], Time(ms), 12
[TOTAL_GC_TIME_%], Time(%), 0.12362212836097661
[READ], Operations, 522
[READ], AverageLatency(us), 1667.4980842911878
[READ], MinLatency(us), 1138
[READ], MaxLatency(us), 14751
[READ], 95thPercentileLatency(us), 2345
[READ], 99thPercentileLatency(us), 3319
[READ], Return=OK, 522
[CLEANUP], Operations, 1
[CLEANUP], AverageLatency(us), 981.0
[CLEANUP], MinLatency(us), 981
[CLEANUP], MaxLatency(us), 981
[CLEANUP], 95thPercentileLatency(us), 981
[CLEANUP], 99thPercentileLatency(us), 981
[UPDATE], Operations, 478
[UPDATE], AverageLatency(us), 17656.142259414228
[UPDATE], MinLatency(us), 14824
[UPDATE], MaxLatency(us), 39135
[UPDATE], 95thPercentileLatency(us), 20815
[UPDATE], 99thPercentileLatency(us), 25439
[UPDATE], Return=OK, 478
```
### workloadb
```
[OVERALL], RunTime(ms), 3012
[OVERALL], Throughput(ops/sec), 332.00531208499336
[TOTAL_GCS_G1_Young_Generation], Count, 1
[TOTAL_GC_TIME_G1_Young_Generation], Time(ms), 11
[TOTAL_GC_TIME_%_G1_Young_Generation], Time(%), 0.36520584329349265
[TOTAL_GCS_G1_Old_Generation], Count, 0
[TOTAL_GC_TIME_G1_Old_Generation], Time(ms), 0
[TOTAL_GC_TIME_%_G1_Old_Generation], Time(%), 0.0
[TOTAL_GCs], Count, 1
[TOTAL_GC_TIME], Time(ms), 11
[TOTAL_GC_TIME_%], Time(%), 0.36520584329349265
[READ], Operations, 945
[READ], AverageLatency(us), 1720.689947089947
[READ], MinLatency(us), 1066
[READ], MaxLatency(us), 16895
[READ], 95thPercentileLatency(us), 2363
[READ], 99thPercentileLatency(us), 5047
[READ], Return=OK, 945
[CLEANUP], Operations, 1
[CLEANUP], AverageLatency(us), 1024.0
[CLEANUP], MinLatency(us), 1024
[CLEANUP], MaxLatency(us), 1024
[CLEANUP], 95thPercentileLatency(us), 1024
[CLEANUP], 99thPercentileLatency(us), 1024
[UPDATE], Operations, 55
[UPDATE], AverageLatency(us), 17875.272727272728
[UPDATE], MinLatency(us), 15720
[UPDATE], MaxLatency(us), 28831
[UPDATE], 95thPercentileLatency(us), 20303
[UPDATE], 99thPercentileLatency(us), 23983
[UPDATE], Return=OK, 55
```
### workloadc
```
[OVERALL], RunTime(ms), 2194
[OVERALL], Throughput(ops/sec), 455.7885141294439
[TOTAL_GCS_G1_Young_Generation], Count, 1
[TOTAL_GC_TIME_G1_Young_Generation], Time(ms), 12
[TOTAL_GC_TIME_%_G1_Young_Generation], Time(%), 0.5469462169553327
[TOTAL_GCS_G1_Old_Generation], Count, 0
[TOTAL_GC_TIME_G1_Old_Generation], Time(ms), 0
[TOTAL_GC_TIME_%_G1_Old_Generation], Time(%), 0.0
[TOTAL_GCs], Count, 1
[TOTAL_GC_TIME], Time(ms), 12
[TOTAL_GC_TIME_%], Time(%), 0.5469462169553327
[READ], Operations, 1000
[READ], AverageLatency(us), 1813.023
[READ], MinLatency(us), 1014
[READ], MaxLatency(us), 15343
[READ], 95thPercentileLatency(us), 2627
[READ], 99thPercentileLatency(us), 6615
[READ], Return=OK, 1000
[CLEANUP], Operations, 1
[CLEANUP], AverageLatency(us), 1014.0
[CLEANUP], MinLatency(us), 1014
[CLEANUP], MaxLatency(us), 1014
[CLEANUP], 95thPercentileLatency(us), 1014
[CLEANUP], 99thPercentileLatency(us), 1014
```
### workloadd
```
[OVERALL], RunTime(ms), 2966
[OVERALL], Throughput(ops/sec), 337.1544167228591
[TOTAL_GCS_G1_Young_Generation], Count, 1
[TOTAL_GC_TIME_G1_Young_Generation], Time(ms), 11
[TOTAL_GC_TIME_%_G1_Young_Generation], Time(%), 0.37086985839514497
[TOTAL_GCS_G1_Old_Generation], Count, 0
[TOTAL_GC_TIME_G1_Old_Generation], Time(ms), 0
[TOTAL_GC_TIME_%_G1_Old_Generation], Time(%), 0.0
[TOTAL_GCs], Count, 1
[TOTAL_GC_TIME], Time(ms), 11
[TOTAL_GC_TIME_%], Time(%), 0.37086985839514497
[READ], Operations, 950
[READ], AverageLatency(us), 1738.6494736842105
[READ], MinLatency(us), 1108
[READ], MaxLatency(us), 16431
[READ], 95thPercentileLatency(us), 2307
[READ], 99thPercentileLatency(us), 4023
[READ], Return=OK, 950
[CLEANUP], Operations, 1
[CLEANUP], AverageLatency(us), 822.0
[CLEANUP], MinLatency(us), 822
[CLEANUP], MaxLatency(us), 822
[CLEANUP], 95thPercentileLatency(us), 822
[CLEANUP], 99thPercentileLatency(us), 822
[INSERT], Operations, 50
[INSERT], AverageLatency(us), 17492.96
[INSERT], MinLatency(us), 14744
[INSERT], MaxLatency(us), 25103
[INSERT], 95thPercentileLatency(us), 22047
[INSERT], 99thPercentileLatency(us), 25103
[INSERT], Return=OK, 50
```
### workloade
```
[OVERALL], RunTime(ms), 4458
[OVERALL], Throughput(ops/sec), 224.31583669807088
[TOTAL_GCS_G1_Young_Generation], Count, 3
[TOTAL_GC_TIME_G1_Young_Generation], Time(ms), 21
[TOTAL_GC_TIME_%_G1_Young_Generation], Time(%), 0.47106325706594887
[TOTAL_GCS_G1_Old_Generation], Count, 0
[TOTAL_GC_TIME_G1_Old_Generation], Time(ms), 0
[TOTAL_GC_TIME_%_G1_Old_Generation], Time(%), 0.0
[TOTAL_GCs], Count, 3
[TOTAL_GC_TIME], Time(ms), 21
[TOTAL_GC_TIME_%], Time(%), 0.47106325706594887
[CLEANUP], Operations, 1
[CLEANUP], AverageLatency(us), 1126.0
[CLEANUP], MinLatency(us), 1126
[CLEANUP], MaxLatency(us), 1126
[CLEANUP], 95thPercentileLatency(us), 1126
[CLEANUP], 99thPercentileLatency(us), 1126
[INSERT], Operations, 12
[INSERT], AverageLatency(us), 17029.0
[INSERT], MinLatency(us), 15528
[INSERT], MaxLatency(us), 19471
[INSERT], 95thPercentileLatency(us), 18159
[INSERT], 99thPercentileLatency(us), 19471
[INSERT], Return=OK, 12
[INSERT], Return=ERROR, 50
[SCAN], Operations, 938
[SCAN], AverageLatency(us), 3913.6588486140727
[SCAN], MinLatency(us), 2003
[SCAN], MaxLatency(us), 20207
[SCAN], 95thPercentileLatency(us), 6199
[SCAN], 99thPercentileLatency(us), 10047
[SCAN], Return=OK, 938
```
### workloadf
```
[OVERALL], RunTime(ms), 11009
[OVERALL], Throughput(ops/sec), 90.83477155054955
[TOTAL_GCS_G1_Young_Generation], Count, 1
[TOTAL_GC_TIME_G1_Young_Generation], Time(ms), 11
[TOTAL_GC_TIME_%_G1_Young_Generation], Time(%), 0.09991824870560449
[TOTAL_GCS_G1_Old_Generation], Count, 0
[TOTAL_GC_TIME_G1_Old_Generation], Time(ms), 0
[TOTAL_GC_TIME_%_G1_Old_Generation], Time(%), 0.0
[TOTAL_GCs], Count, 1
[TOTAL_GC_TIME], Time(ms), 11
[TOTAL_GC_TIME_%], Time(%), 0.09991824870560449
[READ], Operations, 1000
[READ], AverageLatency(us), 1785.298
[READ], MinLatency(us), 1072
[READ], MaxLatency(us), 16703
[READ], 95thPercentileLatency(us), 2481
[READ], 99thPercentileLatency(us), 4759
[READ], Return=OK, 1000
[READ-MODIFY-WRITE], Operations, 497
[READ-MODIFY-WRITE], AverageLatency(us), 19548.24949698189
[READ-MODIFY-WRITE], MinLatency(us), 16496
[READ-MODIFY-WRITE], MaxLatency(us), 45279
[READ-MODIFY-WRITE], 95thPercentileLatency(us), 23359
[READ-MODIFY-WRITE], 99thPercentileLatency(us), 29407
[CLEANUP], Operations, 1
[CLEANUP], AverageLatency(us), 1210.0
[CLEANUP], MinLatency(us), 1210
[CLEANUP], MaxLatency(us), 1210
[CLEANUP], 95thPercentileLatency(us), 1210
[CLEANUP], 99thPercentileLatency(us), 1210
[UPDATE], Operations, 497
[UPDATE], AverageLatency(us), 17749.22334004024
[UPDATE], MinLatency(us), 15288
[UPDATE], MaxLatency(us), 41727
[UPDATE], 95thPercentileLatency(us), 20831
[UPDATE], 99thPercentileLatency(us), 25551
[UPDATE], Return=OK, 497
```

## GO-TPC测试
### tpcc数据准备
受限于集群计算能力，我把warehouses数量限制在100个。
> go-tpc tpcc -H 10.0.0.4 -P 4000 -D tpcc --warehouses 100 prepare
### tpcc测试
> go-tpc tpcc -H 10.0.0.4 -P 4000 -D tpcc --warehouses 100 run
```
DELIVERY - Takes(s): 848.8, Count: 2738, TPM: 193.5, Sum(ms): 1890676, Avg(ms): 690, 95th(ms): 1500, 99th(ms): 4000, 99.9th(ms): 4000
DELIVERY_ERR - Takes(s): 0.0, Count: 3, TPM: 3892.0, Sum(ms): 814, Avg(ms): 271, 95th(ms): 1000, 99th(ms): 1000, 99.9th(ms): 1000
NEW_ORDER - Takes(s): 849.7, Count: 29787, TPM: 2103.3, Sum(ms): 6798222, Avg(ms): 228, 95th(ms): 512, 99th(ms): 1500, 99.9th(ms): 4000
NEW_ORDER_ERR - Takes(s): 0.0, Count: 3, TPM: 3887.0, Sum(ms): 265, Avg(ms): 88, 95th(ms): 128, 99th(ms): 128, 99.9th(ms): 128
ORDER_STATUS - Takes(s): 850.0, Count: 2606, TPM: 184.0, Sum(ms): 36716, Avg(ms): 14, 95th(ms): 40, 99th(ms): 64, 99.9th(ms): 160
PAYMENT - Takes(s): 849.8, Count: 28151, TPM: 1987.6, Sum(ms): 4775875, Avg(ms): 169, 95th(ms): 256, 99th(ms): 512, 99.9th(ms): 2000
PAYMENT_ERR - Takes(s): 0.0, Count: 4, TPM: 5184.2, Sum(ms): 134, Avg(ms): 33, 95th(ms): 64, 99th(ms): 64, 99.9th(ms): 64
STOCK_LEVEL - Takes(s): 849.7, Count: 2697, TPM: 190.4, Sum(ms): 62555, Avg(ms): 23, 95th(ms): 48, 99th(ms): 96, 99.9th(ms): 512
```
### Grafana Dashboard截图
![TiDB Summary QPS截图](resource/homework_2_tidb_tpcc_qps.png)
![TiDB Summary Duration截图](resource/homework_2_tidb_tpcc_duration.png)
![TiKV Details Server CPU截图](resource/homework_2_tikv_tpcc_cpu.png)
![TiKV Details Server QPS截图](resource/homework_2_tikv_tpcc_qps.png)
![TiKV Details gRPS QPS截图](resource/homework_2_tikv_tpcc_grpc_qps.png)
![TiKV Details gRPS duration截图](resource/homework_2_tikv_tpcc_grpc_duration.png)
### tpch数据准备
受限于集群计算能力，我把scale factor设置为4。
> go-tpc tpch -H 10.0.0.4 -P 4000 -D tpch --sf 4 prepare
### tpch测试结果
> go-tpc tpch -H 10.0.0.4 -P 4000 -D tpch --sf 4 run
```
Q1     - Takes(s): 997.3, Count: 1, TPM: 0.1, Sum(ms): 100450, Avg(ms): 100450, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q10    - Takes(s): 987.6, Count: 1, TPM: 0.1, Sum(ms): 110209, Avg(ms): 110209, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q10_ERR - Takes(s): 0.4, Count: 1, TPM: 153.1, Sum(ms): 988516, Avg(ms): 988516, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q11    - Takes(s): 1058.4, Count: 1, TPM: 0.1, Sum(ms): 39344, Avg(ms): 39344, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q11_ERR - Takes(s): 0.1, Count: 1, TPM: 630.9, Sum(ms): 987481, Avg(ms): 987481, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q12    - Takes(s): 999.4, Count: 1, TPM: 0.1, Sum(ms): 98358, Avg(ms): 98358, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q12_ERR - Takes(s): 0.1, Count: 1, TPM: 630.3, Sum(ms): 1058347, Avg(ms): 1058347, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q13    - Takes(s): 1073.3, Count: 1, TPM: 0.1, Sum(ms): 24510, Avg(ms): 24510, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q13_ERR - Takes(s): 0.5, Count: 1, TPM: 132.2, Sum(ms): 998878, Avg(ms): 998878, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q14    - Takes(s): 996.0, Count: 1, TPM: 0.1, Sum(ms): 101801, Avg(ms): 101801, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q14_ERR - Takes(s): 0.3, Count: 1, TPM: 173.8, Sum(ms): 1072933, Avg(ms): 1072933, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q15    - Takes(s): 983.2, Count: 1, TPM: 0.1, Sum(ms): 114570, Avg(ms): 114570, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q15_ERR - Takes(s): 0.3, Count: 1, TPM: 238.8, Sum(ms): 995736, Avg(ms): 995736, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q16    - Takes(s): 1067.4, Count: 1, TPM: 0.1, Sum(ms): 30409, Avg(ms): 30409, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q16_ERR - Takes(s): 0.5, Count: 1, TPM: 132.2, Sum(ms): 982719, Avg(ms): 982719, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q17_ERR - Takes(s): 0.3, Count: 1, TPM: 201.0, Sum(ms): 1067086, Avg(ms): 1067086, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q2     - Takes(s): 1066.8, Count: 1, TPM: 0.1, Sum(ms): 30955, Avg(ms): 30955, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q2_ERR - Takes(s): 0.4, Count: 1, TPM: 153.0, Sum(ms): 996949, Avg(ms): 996949, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q3     - Takes(s): 975.8, Count: 1, TPM: 0.1, Sum(ms): 121972, Avg(ms): 121972, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q3_ERR - Takes(s): 0.1, Count: 1, TPM: 426.5, Sum(ms): 1066694, Avg(ms): 1066694, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q4     - Takes(s): 982.4, Count: 1, TPM: 0.1, Sum(ms): 115407, Avg(ms): 115407, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q4_ERR - Takes(s): 0.4, Count: 1, TPM: 153.5, Sum(ms): 975423, Avg(ms): 975423, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q5_ERR - Takes(s): 0.4, Count: 2, TPM: 305.9, Sum(ms): 2079525, Avg(ms): 1039762, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q6     - Takes(s): 1001.4, Count: 1, TPM: 0.1, Sum(ms): 96364, Avg(ms): 96364, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q7     - Takes(s): 993.9, Count: 1, TPM: 0.1, Sum(ms): 103866, Avg(ms): 103866, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q7_ERR - Takes(s): 0.1, Count: 1, TPM: 423.1, Sum(ms): 1001287, Avg(ms): 1001287, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q8     - Takes(s): 976.2, Count: 1, TPM: 0.1, Sum(ms): 121631, Avg(ms): 121631, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q8_ERR - Takes(s): 0.1, Count: 1, TPM: 425.8, Sum(ms): 993785, Avg(ms): 993785, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q9     - Takes(s): 989.1, Count: 1, TPM: 0.1, Sum(ms): 108723, Avg(ms): 108723, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
Q9_ERR - Takes(s): 0.5, Count: 1, TPM: 132.1, Sum(ms): 975705, Avg(ms): 975705, 95th(ms): 16000, 99th(ms): 16000, 99.9th(ms): 16000
```
![TiDB Summary QPS截图](resource/homework_2_tidb_tpch_qps.png)
![TiDB Summary Duration截图](resource/homework_2_tidb_tpch_duration.png)
![TiKV Details Server CPU截图](resource/homework_2_tikv_tpch_cpu.png)
![TiKV Details Server QPS截图](resource/homework_2_tikv_tpch_qps.png)
![TiKV Details gRPS QPS截图](resource/homework_2_tikv_tpch_grpc_qps.png)
![TiKV Details gRPS duration截图](resource/homework_2_tikv_tpch_grpc_duration.png)

### 总结
CPU再次成为我测试集群的瓶颈。不同于之前的测试，CPU在TiDB和TiKV节点都保持在很高的使用率。测试数据准备的过程消耗非常多的时间，我认为这是受制于内存和IO的限制。在这个测试的过程中，TiDB节点还出现了一次死机的情形，Azure尝试修复没有成功，最后只能重启。后来检查原因应该是CPU负载太高，无法响应服务.
![CPU负载100%无法](resource/homework_2_tpc_no_response.png)
