# Solr Search for Collections manager #

This repository contains the configuration and setup of the Solr instance used by the DINA Collections manager.

## Development ##

**tldr;**

* Clone repository
* Create a new branch and make changes
* Create a pull request for review
* Merge pull request
* Pull new image from [Dockerhub](https://hub.docker.com/r/dina/collections-solr)

### Configuration files ###

The main part of the Solr index is the configuration files. It consists of two files, one for managing the connection to the datasource and one for defining the schema.

#### Datasource config ####

The connection to the datasource is defined in [data-config.xml](conf/data-config.xml). It uses a MySQL-connector and defines a set of entities.

Each entity contains three different queries:

* **query**: Query to perform full import. Used when Solr instance is restarted and needs to do a full index.
* **deltaQuery**: Query to fetch IDs of all entites created after last import.
* **deltaImportQuery**: Query to fetch data for all IDs from the *deltaQuery*.

The fields of each entity is prefixed to be able to separate the from each other. For exmaple `Collecting event` fields are prefixed with `ce_{some_attribute}`.

All entites should also contain a field for:

* **entity_type**: The type of entity, used to search for specific types. Eg. `collectingevent` or `locality`.
* **primary_id**: The primary id of the entity unmodified. Used to for example fetch ore data from Collections API.
* **id**: Unique ID for the Solr index. Usally the *primary_id* prefixed using the same prefix as before. Eg. `ce_{primary_id}` for `Collecting event`.

Fields that are common accross many entites are not prefixed. Such as `DisciplineID`.

#### Schema configuration ####

The DINA schema configuration is located in [dina-schema.xml](conf/dina-schema.xml). The schema consists of the same fields as the [data-config.xml](conf/data-config.xml) but with some other attributes.

All fields should have the `stored` attribute set to `false` to not store the actual value for entities. This way you can only fetch primary ids and then get more data by doing a lookup in the Collection API.

The configartion sets the default join operator to `AND` in the `solrQueryParser` configuration.

### Building ###

The Solr index is build into a Docker image and pushed to [Dockerhub](https://hub.docker.com/r/dina/collections-solr) using [Travis-CI](https://travis-ci.org/DINA-Web/collections-solr). The configuration for the image is found in the [Dockerfile](Dockerfile) and the Travis configuration is located in [.travis.yml](.travis.yml).
