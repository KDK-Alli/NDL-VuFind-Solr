# NDL-VuFind-Solr

Solr 6 for NDL-VuFind2 (Finna 2)

This is for the most parts vanilla Solr 6 with VuFind schemas. The following changes have been made:

- Solr distribution is in vendor directory
- Solr home (set in solr.in.finna.sh[.sample]) is ./vufind which contains the Finna VuFind core configs
- The JTS libraries from http://tsusiatsoftware.net/jts/main.html have been added to vendor/server/solr-webapp/webapp/WEB-INF/lib (putting them in vufind/lib doesn't seem to work, probably because of SOLR-4852 and SOLR-6188, and trying workaround still doesn't let JTS load properly):
  - jts
  - jtsio
- The following libraries are copied to vufind/lib (having them in solrconfig.xml doesn't play nice with dynamic collection management in SolrCloud):
  - vendor/contrib/analysis-extras/lib/icu4j-*.jar
  - vendor/contrib/analysis-extras/lucene-libs/lucene-analyzers-icu-*.jar
- The vendor/docs directory has been removed
- Voikko is used for Finnish language processing

## Installation

### Prerequisites

1. Install libvoikko and the dictionary
    - For CentOS 6 see the first three steps at https://github.com/NatLibFi/SolrPlugins/wiki/Voikko-plugin
    - For CentOS 7:

            yum install libvoikko
            cd /tmp
            wget http://www.puimula.org/htp/testing/voikko-snapshot/dict-morphoid.zip
            mkdir /etc/voikko
            cd /etc/voikko
            unzip /tmp/dict-morphoid.zip
            rm /tmp/dict-morphoid.zip

### Solr

1. Put the files somewhere
2. Add user solr
3. chown the files and directories to solr user
4. Copy vufind/solr.in.finna.sh.sample to vufind/solr.in.finna.sh and edit as required
5. Use the following command to start Solr manually:

        SOLR_INCLUDE=vufind/solr.in.finna.sh vendor/bin/solr start

6. To enable startup via system init and management with service command in init-based systems like RHEL 6.x, copy vufind/solr.finna-init-script to file /etc/init.d/solr, make it executable, change the paths in it and execute the following commands:

        chkconfig --add solr
        chkconfig solr on

7. With systemd-based systemd, like CentOS 7, copy vufind/solr.service to /etc/systemd/system/, change paths in it and execute the following commands:

        systemctl daemon-reload
        systemctl enable solr

8. In init-based systems, start Solr with command:

        service solr start

9. In systemd-based systems, start Solr with command:

        systemctl start solr

10. Check the logs at vufind/logs for any errors

11. If running in solrcloud mode, use the following command to add a core configuration to Zookeeper:

        vendor/server/scripts/cloud-scripts/zkcli.sh -zkhost localhost:9983 -cmd upconfig -d vufind/biblio/conf/ -n biblio3

    Then you can create a new collection that uses the configuration by calling the collections API:

        curl 'http://localhost:8983/solr/admin/collections?action=CREATE&name=biblio3&numShards=1&replicationFactor=3&collection.configName=biblio3'

    Use an alias to point to the current index version in use. This way you can just point the alias to a new index version when it's ready to use:

        curl "http://localhost:8983/solr/admin/collections?action=CREATEALIAS&name=biblioprod&collections=biblio3"

    When a collection is no longer needed, remove it using the collections API:

        curl 'http://localhost:8983/solr/admin/collections?action=DELETE&name=biblio2'

