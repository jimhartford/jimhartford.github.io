---
layout: post
title: "Zend_Date vs DateTime Performance"
date: 2009-03-06 09:03:51
categories: performance php
author: Jim Hartford
---

After running into some issues with slowness on a project I am working on using the Zend Framework I started trying 
to hunt down the issue. In this case, the problem was being caused by using Zend_Date. Here are the results of my tests:

ab Results of PHP's DateTime

~~~
#ab -n 100 -c 10 http://localhost/test/datebm_php.php
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient).....done


Server Software:        Apache/1.3.37
Server Hostname:        localhost
Server Port:            80

Document Path:          /test/datebm_php.php
Document Length:        22 bytes

Concurrency Level:      10
Time taken for tests:   0.163 seconds
Complete requests:      100
Failed requests:        0
Write errors:           0
Total transferred:      26100 bytes
HTML transferred:       2200 bytes
Requests per second:    613.44 [#/sec] (mean)
Time per request:       16.301 [ms] (mean)
Time per request:       1.630 [ms] (mean, across all concurrent requests)
Transfer rate:          156.36 [Kbytes/sec] received

Connection Times (ms)
             min  mean[+/-sd] median   max
Connect:        0    1   0.4      1       3
Processing:     5   15   5.2     13      37
Waiting:        4   15   5.2     13      37
Total:          5   16   5.3     14      38

Percentage of the requests served within a certain time (ms)
 50%     14
 66%     15
 75%     18
 80%     21
 90%     25
 95%     27
 98%     30
 99%     38
100%     38 (longest request)
~~~

ab Results using Zend_Date

~~~
#ab -n 100 -c 10 http://localhost/test/datebm_zend.php
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient).....done


Server Software:        Apache/1.3.37
Server Hostname:        localhost
Server Port:            80

Document Path:          /test/datebm_zend.php
Document Length:        22 bytes

Concurrency Level:      10
Time taken for tests:   6.724 seconds
Complete requests:      100
Failed requests:        0
Write errors:           0
Total transferred:      26100 bytes
HTML transferred:       2200 bytes
Requests per second:    14.87 [#/sec] (mean)
Time per request:       672.386 [ms] (mean)
Time per request:       67.239 [ms] (mean, across all concurrent requests)
Transfer rate:          3.79 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   0.3      1       2
Processing:    87  648 157.0    663    1502
Waiting:       87  647 154.6    663    1502
Total:         88  649 156.9    664    1503

Percentage of the requests served within a certain time (ms)
  50%    664
  66%    677
  75%    697
  80%    708
  90%    757
  95%    785
  98%   1187
  99%   1503
 100%   1503 (longest request)
~~~
 
As you can see, using Zend\_Date is significantly slower going from 613.44 requests per second with PHP's DateTime
to only 14.87 requests per second with Zend\_Date

 Below is the code that was used
 
 datebm_php.php
 
~~~
 <?php 
 date_default_timezone_set('America/New_York');
 
 $dbDate = date("Y-m-d H:i:s");
 $date = new DateTime($dbDate);
 echo $date->format('m/d/Y h:iA');
~~~
{: .language-php}

 datebm_zend.php 
 
~~~
 <?php  
 date_default_timezone_set('America/New_York');
 
 function __autoload ($class)
 {
     $file = str_replace("_", "/", $class) . ".php";
     include_once($file);
 }
 
 $dbDate = date("Y-m-d H:i:s");
 Zend_Date::setOptions(array('format_type' => 'php'));
 $date = new Zend_Date($dbDate, Zend_Date::ISO_8601);
 echo $date->get('m/d/Y h:iA');
~~~
{: .language-php}
 

 
 
 
 
 