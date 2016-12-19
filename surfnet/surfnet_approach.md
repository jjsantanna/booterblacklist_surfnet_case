# SURFnet's approach

SURFnet developed a plugin to monitor DNS records available at https://github.com/SURFnet/eemo. The plugin is called "dnsqrlog". To conigure this plugin you just need (1) clone the github repositorio, (2) include the IP address that you want to monitor and (3) downloand a Booter blacklist at http://booterblacklist.com.

The configuration of the EEMO plugin you just need to create a ".conf" file into folder "config" that contains the configuration of module "dnsqrlog". Following there is one example for you to take a look on.


```
...

modules:
{
        dnsqrlog:
        {
                lib = "/usr/local/lib/libeemo_dnsqrlog.so";

                modconf:
                {
                        #Add the IPs (v4 and v6) that you want to monitor
                        listen_ips = [ "192.168.0.0.1", "127.0.0.1", "255.255.255.255" ]; 
                           
                        #Add the Booter blacklist here
                        log_domains =
                        [
                                "absolut-stresser.net.",
                                "acidstresser.net.",
                                "agonyproducts.com.",
                                ...
                        ];

                log_file = "booter-queries.txt";
                };
        };
};

...
```

Note that you should change the "list_ips" accordingly to the IP address that you want to monitor and the "log_domains" to include the booter blacklist. Finally, and **very important** you must to anonymize EEMO's output ("booter-queries.txt") before send to us! So, you can simply download the example in https://github.com/jjsantanna/IP_anonymization_by_Roland and run the C code.