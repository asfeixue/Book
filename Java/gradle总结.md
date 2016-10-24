gradle总结


gradle 驱动 sonar

gradle Multi-Project中，在主build.gradle中定义如下结构：

    plugins {
        id "org.sonarqube" version "2.2" apply false
        id "jacoco"
    }
    subprojects {
        apply plugin: "org.sonarqube"
        apply plugin: "jacoco"
        sonarqube {
            properties {
                property "sonar.host.url", "xxxxxx"
                property "sonar.verbose", true
            }
        }
        dependencies {
            provided "org.sonarsource.scanner.gradle:sonarqube-gradle-plugin:2.2"
            provided "org.jacoco:jacoco-maven-plugin:0.7.4.201502262128"
        }
    }

需要gradle 3.1
配置：【id "org.sonarqube" version "2.2" apply false】中的"apply false"在gradle2.8中暂不支持。