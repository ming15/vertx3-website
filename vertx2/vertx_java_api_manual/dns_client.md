# DNS Client

Often you will find yourself in situations where you need to obtain DNS informations in an asynchronous fashion. Unfortunally this is not possible with the API that is shipped with Java itself. Because of this Vert.x offers it's own API for DNS resolution which is fully asynchronous.

To obtain a DnsClient instance you will create a new via the Vertx instance.

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53), new InetSocketAddress("10.0.0.2", 53));
Be aware that you can pass in a varargs of InetSocketAddress arguments to specifiy more then one DNS Server to try to query for DNS resolution. The DNS Servers will be queried in the same order as specified here. Where the next will be used once the first produce an error while be used.

lookup

Try to lookup the A (ipv4) or AAAA (ipv6) record for a given name. The first which is returned will be used, so it behaves the same way as you may be used from when using "nslookup" on your operation system.

To lookup the A / AAAA record for "vertx.io" you would typically use it like:

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53));
client.lookup("vertx.io", new AsyncResultHandler<InetAddress>() {
    public void handle(AsyncResult<InetAddress> ar) {
        if (ar.succeeded()) {
            System.out.println(ar.result());
        } else {
            log.error("Failed to resolve entry", ar.cause());
        }
    }
});
Be aware that it either would use an Inet4Address or Inet6Address in the AsyncResult depending on if an A or AAAA record was resolved.

lookup4

Try to lookup the A (ipv4) record for a given name. The first which is returned will be used, so it behaves the same way as you may be used from when using "nslookup" on your operation system.

To lookup the A record for "vertx.io" you would typically use it like:

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53));
client.lookup4("vertx.io", new AsyncResultHandler<Inet4Address>() {
    public void handle(AsyncResult<Inet4Address> ar) {
        if (ar.succeeded()) {
            System.out.println(ar.result());
        } else {
            log.error("Failed to resolve entry", ar.cause());
        }
    }
});
As it only resolves A records and so is ipv4 only it will use Inet4Address as result.

lookup6

Try to lookup the AAAA (ipv5) record for a given name. The first which is returned will be used, so it behaves the same way as you may be used from when using "nslookup" on your operation system.

To lookup the A record for "vertx.io" you would typically use it like:

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53));
client.lookup6("vertx.io", new AsyncResultHandler<Inet6Address>() {
    public void handle(AsyncResult<Inet6Address> ar) {
        if (ar.succeeded()) {
            System.out.println(ar.result());
        } else {
            log.error("Failed to resolve entry", ar.cause());
        }
    }
});
As it only resolves AAAA records and so is ipv6 only it will use Inet6Address as result.

resolveA

Try to resolve all A (ipv4) records for a given name. This is quite similar to using "dig" on unix like operation systems.

To lookup all the A records for "vertx.io" you would typically do:

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53));
client.resolveA("vertx.io", new AsyncResultHandler<List<Inet4Address>>() {
    public void handle(AsyncResult<List<Inet4Address>> ar) {
        if (ar.succeeded()) {
            List<Inet4Address> records = ar.result());
            for (Inet4Address record: records) {
                System.out.println(record);
            }
        } else {
            log.error("Failed to resolve entry", ar.cause());
        }
    }
});
As it only resolves A records and so is ipv4 only it will use Inet4Address as result.

resolveAAAA

Try to resolve all AAAA (ipv6) records for a given name. This is quite similar to using "dig" on unix like operation systems.

To lookup all the AAAAA records for "vertx.io" you would typically do:

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53));
client.resolveAAAA("vertx.io", new AsyncResultHandler<List<Inet6Address>>() {
    public void handle(AsyncResult<List<Inet6Address>> ar) {
        if (ar.succeeded()) {
            List<Inet6Address> records = ar.result());
            for (Inet6Address record: records) {
                System.out.println(record);
            }
        } else {
            log.error("Failed to resolve entry", ar.cause());
        }
    }
});
As it only resolves AAAA records and so is ipv6 only it will use Inet6Address as result.

resolveCNAME

Try to resolve all CNAME records for a given name. This is quite similar to using "dig" on unix like operation systems.

To lookup all the CNAME records for "vertx.io" you would typically do:

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53));
client.resolveCNAME("vertx.io", new AsyncResultHandler<List<String>>() {
    public void handle(AsyncResult<List<String>> ar) {
        if (ar.succeeded()) {
            List<String> records = ar.result());
            for (String record: records) {
                System.out.println(record);
            }
        } else {
            log.error("Failed to resolve entry", ar.cause());
        }
    }
});
resolveMX

Try to resolve all MX records for a given name. The MX records are used to define which Mail-Server accepts emails for a given domain.

To lookup all the MX records for "vertx.io" you would typically do:

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53));
client.resolveMX("vertx.io", new AsyncResultHandler<List<MxRecord>>() {
    public void handle(AsyncResult<List<MxRecord>> ar) {
        if (ar.succeeded()) {
            List<MxRecord> records = ar.result());
            for (MxRecord record: records) {
                System.out.println(record);
            }
        } else {
            log.error("Failed to resolve entry", ar.cause());
        }
    }
});
Be aware that the List will contain the MxRecords sorted by the priority of them, which means MxRecords with smaller priority coming first in the List.

The MxRecord allows you to access the priority and the name of the MX record by offer methods for it like:

MxRecord record = ...
record.priority();
record.name();
resolveTXT

Try to resolve all TXT records for a given name. TXT records are often used to define extra informations for a domain.

To resolve all the TXT records for "vertx.io" you could use something along these lines:

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53));
client.resolveTXT("vertx.io", new AsyncResultHandler<List<String>>() {
    public void handle(AsyncResult<List<String>> ar) {
        if (ar.succeeded()) {
            List<String> records = ar.result());
            for (String record: records) {
                System.out.println(record);
            }
        } else {
            log.error("Failed to resolve entry", ar.cause());
        }
    }
});
resolveNS

Try to resolve all NS records for a given name. The NS records specify which DNS Server hosts the DNS informations for a given domain.

To resolve all the NS records for "vertx.io" you could use something along these lines:

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53));
client.resolveNS("vertx.io", new AsyncResultHandler<List<String>>() {
    public void handle(AsyncResult<List<String>> ar) {
        if (ar.succeeded()) {
            List<String> records = ar.result());
            for (String record: records) {
                System.out.println(record);
            }
        } else {
            log.error("Failed to resolve entry", ar.cause());
        }
    }
});
resolveSRV

Try to resolve all SRV records for a given name. The SRV records are used to define extra informations like port and hostname of services. Some protocols need this extra informations.

To lookup all the SRV records for "vertx.io" you would typically do:

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53));
client.resolveMX("vertx.io", new AsyncResultHandler<List<SrvRecord>>() {
    public void handle(AsyncResult<List<SrvRecord>> ar) {
        if (ar.succeeded()) {
            List<SrvRecord> records = ar.result());
            for (SrvRecord record: records) {
                System.out.println(record);
            }
        } else {
            log.error("Failed to resolve entry", ar.cause());
        }
    }
});
Be aware that the List will contain the SrvRecords sorted by the priority of them, which means SrvRecords with smaller priority coming first in the List.

The SrvRecord allows you to access all informations contained in the SRV record itself:

SrvRecord record = ...
record.priority();
record.name();
record.priority();
record.weight();
record.port();
record.protocol();
record.service();
record.target();
Please refer to the API docs for the exact details.

resolvePTR

Try to resolve the PTR record for a given name. The PTR record maps an ipaddress to a name.

To resolve the PTR record for the ipaddress 10.0.0.1 you would use the PTR notion of "1.0.0.10.in-addr.arpa"

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53));
client.resolveTXT("1.0.0.10.in-addr.arpa", new AsyncResultHandler<String>() {
    public void handle(AsyncResult<String> ar) {
        if (ar.succeeded()) {
            String record = ar.result());
            System.out.println(record);
        } else {
            log.error("Failed to resolve entry", ar.cause());
        }
    }
});
reverseLookup

Try to do a reverse lookup for an ipaddress. This is basically the same as resolve a PTR record, but allows you to just pass in the ipaddress and not a valid PTR query string.

To do a reverse lookup for the ipaddress 10.0.0.1 do something similar like this:

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53));
client.reverseLookup("10.0.0.1", new AsyncResultHandler<String>() {
    public void handle(AsyncResult<String> ar) {
        if (ar.succeeded()) {
            String record = ar.result());
            System.out.println(record);
        } else {
            log.error("Failed to resolve entry", ar.cause());
        }
    }
});
Error handling

As you saw in previous sections the DnsClient allows you to pass in a Handler which will be notified with an AsyncResult once the query was complete. In case of an error it will be notified with a DnsException which will hole a DnsResponseCode that indicate why the resolution failed. This DnsResponseCode can be used to inspect the cause in more detail.

Possible DnsResponseCodes are:

NOERROR

No record was found for a given query

FORMERROR

Format error

SERVFAIL

Server failure

NXDOMAIN

Name error

NOTIMPL

Not implemented by DNS Server

REFUSED

DNS Server refused the query

YXDOMAIN

Domain name should not exist

YXRRSET

Resource record should not exist

NXRRSET

RRSET does not exist

NOTZONE

Name not in zone

BADVER

Bad extension mechanism for version

BADSIG

Bad signature

BADKEY

Bad key

BADTIME

Bad timestamp

All of those errors are "generated" by the DNS Server itself.

You can obtain the DnsResponseCode from the DnsException like:

DnsClient client = vertx.createDnsClient(new InetSocketAddress("10.0.0.1", 53));
client.lookup("nonexisting.vert.xio", new AsyncResultHandler<InetAddress>() {
    public void handle(AsyncResult<InetAddress> ar) {
        if (ar.succeeded()) {
            InetAddress record = ar.result());
            System.out.println(record);
        } else {
            Throwable cause = ar.cause();
            if (cause instanceof DnsException) {
                DnsException exception = (DnsException) cause;
                DnsResponseCode code = exception.code();
                ...
            } else {
                log.error("Failed to resolve entry", ar.cause());
            }
        }
    }
});
