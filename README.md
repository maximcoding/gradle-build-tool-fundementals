# Gradle jooq codegen

```
buildscript {
    repositories {
        mavenLocal()
        mavenCentral()
    }
    dependencies {
        classpath 'org.jooq:jooq-codegen:3.16.2'
        classpath 'mysql:mysql-connector-java:8.0.32'
    }
}

plugins {
    id 'nu.studer.jooq' version '8.1'
}

group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
    maven {
        credentials {
            username = "${artifactory_user}"
            password = "${artifactory_password}"
        }
        url "${artifactory_contextUrl}/gradle-dev-local"
        allowInsecureProtocol true
    }
}

dependencies {
    // JOOQ & MySQL
    implementation group: 'mysql', name: 'mysql-connector-java', version: '8.0.32'
    implementation group: 'org.jooq', name: 'jooq', version: '3.16.2'
    implementation group: 'org.jooq', name: 'jooq-meta', version: '3.16.2'
    compileOnly group: 'org.jooq', name: 'jooq-codegen', version: '3.16.2'
    jooqGenerator("mysql:mysql-connector-java:8.0.32")
}

test {
    useJUnitPlatform()
}

jooq {
    def props = new Properties()
    file("gradle.properties").withInputStream {
        props.load(it)
    }
    version = jooqGeneratorVersion // the default
    edition = nu.studer.gradle.jooq.JooqEdition.OSS  // the default
    configurations {
        main {  // name of the jOOQ configuration
            generateSchemaSourceOnCompilation = true  // default
            generationTool {
                jdbc {
                    driver = 'com.mysql.cj.jdbc.Driver'
                    url = props['databaseUrl']
                    user = props['databaseUser']
                    password = props['databasePass']
                }
                generator {
                    name = 'org.jooq.codegen.DefaultGenerator'
                    database {
                        name = 'org.jooq.meta.mysql.MySQLDatabase'
                        schemata {
                            def schemas = props['databaseSchemas'].split(',')
                            for (item in schemas) {
                                schema {
                                    inputSchema = item
                                }
                            }
                        }
                    }
                    generate {
                        deprecated = false
                        records = true
                    }
                    target {
                        packageName = 'nu.studer.jooq-generated'
                        directory = props['jooqGenerateDir']  // default
                    }
                    strategy.name = 'org.jooq.codegen.DefaultGeneratorStrategy'
                }
            }
        }
    }
}
