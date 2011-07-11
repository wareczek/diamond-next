About
=====

Diamond is a python daemon that collects system metrics and publishes them to Graphite. It is 
capable of collecting cpu, memory, network, i/o, load and disk metrics.  Additionally, 
it features an API for implementing custom collectors for gathering metrics from almost any source. 

Installation
=====

**Dependencies**
-   CentOS or Ubuntu
-   Python 2.4+
-   python-configobj
-   pysnmp
-   pyasn1
-   ant (for packaging)

Usage 
=====

For testing, diamond can be invoked via ant: 

> ant run

The *run* task will invoke diamond in debug mode for testing.

Ant can also build packages for CentOS and Ubuntu.

> ant package
> sudo dpkg -i build-debian/diamond-2.0.0/diamond-2.0.0-0.deb

The *package* task will detect Ubuntu or CentOS and build .debs or .rpms. 
 
> ant tar

The *tar* task will build a tarball if thats your thing.

Configuration
=====

If you've installed diamond via a package, the configuration file is /etc/diamond/diamond.cfg. By default, diamond 
will push to a graphite server host "diamond". You should probably change this to point to your own graphite server.

Other configuration should not be necessary but feel free to so.

By default diamond publishes metrics using the following form:

systems.<hostname>.<metrics>.<metric>

You can override the "systems" portion of the metric path by changing the "path_prefix" setting in the configuration file. 

Collectors
===== 

System Collectors
======

-   NetworkCollector
-   CPUCollector
-   MemoryCollector
-   LoadAverageCollector
-   IOCollector
-   VMStatCollector
-   DiskSpaceCollector
-   TCPStatsCollector

Custom Collectors
======

Diamond collectors run within the diamond process under the diamond server process and collect metrics that can be published to a graphite server.

Collectors are subclasses of diamond.collector.Collector. In their simplest form, they need to implement a single method called "collect".

> import diamond.collector
>
> class ExampleCollector(diamond.collector.Collector):
>    
>    def collect(self):
>        """
>        Overrides the Collector.collect method
>        """
>        # Set Metric Name
>        metric_name = "my.example.metric"
>
>        # Set Metric Value
>        metric_value = 42
>
>        # Publish Metric
>        self.publish(metric_name, metric_value)

To run this collector in test mode you can invoke the diamond server with the -r option and specify the collector path.

> python src/diamond/server.py -f -v -r examples/examplecollector.py

Diamond supports dynamic addition of collectors. Its configured to scan for new collectors on a regular interval (configured in diamond.cfg). 
If diamond detects a new collector, or that a collectors module has changed (based on the file's last modified timestamp), it will be reloaded.

Diamond looks for collectors in /usr/lib/diamond/collectors/ (on Ubuntu). By default diamond will invoke the "collect" method every 60 seconds. 

Diamond collectors that require a separate configuration file should place a .cfg file in /etc/diamond/. 
The configuration file name should match the name of the diamond collector class.  For example, a collector called 
"examplecollector.ExampleCollector" could have its configuration file placed in /etc/diamond/ExampleCollector.cfg.
