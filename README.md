


# Configurando o Jacoco

Primeiro, adicione ao build.gradle de nível de projeto:

```groovy
buildscript {
	 dependencies {
	 ...
	 classpath 'org.jacoco:org.jacoco.core:0.8.5'
	 }
 }
 ```


Em seguida crie um arquivo chamado `./jacoco.gradle` e adicione essas linhas a ela:
```groovy
project.afterEvaluate {
  def flavors = android.productFlavors.collect { flavor -> flavor.name }

  // If there are no flavors defined, add empty flavor.
  if (!flavors) flavors.add('')

  def buildType = "Debug"
  flavors.each { productFlavorName ->
	  def taskNameWithFlavor, pathNameWithFlavor
        if (!productFlavorName) {
            //empty flavors
		  taskNameWithFlavor = pathNameWithFlavor = "${buildType}"
	    }
	    else {
          taskNameWithFlavor = "${productFlavorName}${buildType}"
		  pathNameWithFlavor = "${productFlavorName}${buildType}"
	    }
        def unitTestsTaskName = "test${taskNameWithFlavor.capitalize()}UnitTest"
		def uiTestsTaskName = "connected${taskNameWithFlavor.capitalize()}AndroidTest"

		 // Create coverage task ex: 'jacocoTestReport<Flavor>' depending on
		 // 'testFlavorDebugUnitTest - unit tests' & connectedFlavorDebugAndroidTest - integration tests.
		 task "jacocoTestReport${productFlavorName.capitalize()}"(type: JacocoReport, dependsOn: [unitTestsTaskName, uiTestsTaskName]) {
			 group = "Reporting"
			 description = "Generate Jacoco coverage reports on the ${taskNameWithFlavor.capitalize()} build. Flavor: $pathNameWithFlavor"

			  classDirectories = fileTree(
                    dir: "${project.buildDir}/tmp/kotlin-classes/$pathNameWithFlavor",
                    excludes: ['**/domain/**',
                            '**/di/**',
                            '**/gps/**'
  ])

            def coverageSourceDirs = [
                    "src/main/java",
                    "src/$productFlavorName/java",
                    "src/$buildType/java"
  ]
            additionalSourceDirs = files(coverageSourceDirs)
            sourceDirectories = files(coverageSourceDirs)
            executionData = fileTree(dir: project.projectDir, includes: ["**/*.exec", "**/*.ec"])

            reports {
			  xml.enabled = true
			  html.enabled = true
			}
		 }
	 }
 }
```

Depois adicione no build.gradle do seu módulo principal/base (normalmente chamado de :app), as seguintes linhas abaixo do apply dos outros plugins:

```groovy
apply plugin: 'jacoco'
apply from: './jacoco.gradle'
```
Com essa configuração você pode rodar o cover de teste unitário e visualizar no arquivo index.html na pasta app/build/reports/jacoco/jacocoTest{nome da sua flavor}UnitTestReport/html/index.html
```bash
./gradlew jacocoTestReport{nome da sua flavor}
 ```

![enter image description here](https://raw.githubusercontent.com/GaldinoJr/TesteJacocoSonar/master/app/src/main/res/drawable-v24/Captura%20de%20Tela%202020-04-01%20%C3%A0s%2020.14.58.png)

# Adicione o plugin do SonarQube
  Primeiro, adicione ao build.gradle de nível de projeto:

 ```
 groovy repositories {
	 ...
	 maven {
	 url "https://plugins.gradle.org/m2/"
	 }
}
 ```

```groovy
buildscript {
	 dependencies {
	 ..
	 classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.4.RELEASE")
	 classpath "org.sonarsource.scanner.gradle:sonarqube-gradle-plugin:2.8"
	 }
}
 ```

```groovy
allprojects {
	maven {
		url "https://plugins.gradle.org/m2/"
	}
}
```

Em seguida crie um arquivo chamado `./sonarqube.gradle` e adicione essas linhas a ela:
obs.: ver o caminho do .xml gerado pelo jacoco (muda conforme a flavor), e configure corretamente os itens: sonar.coverage.jacoco.xmlReportPaths e sonar.junit.reportsPath

```groovy
sonarqube {
  properties {
  property "sonar.exclusions",
                '**/MyApplication**,' +
                        '**/domain/**,' +
                        '**/di/**,' +
                        '**/gps/**,'

  property "sonar.projectName", "App-Vidalink-Android-TESTE-4"
  property "sonar.projectKey", "App-Vidalink-Android-TESTE-4"
  property "sonar.host.url", "http://localhost:9000"
  // retirar para rodar local
//            property "sonar.host.url", "http://35.247.249.42:9000/"
//            property "sonar.login", "ad582835bec9886be86a90fe7b7b31d556795e6a"
//            property "sonar.password", ""

//            property "sonar.scm.provider", "git"
  property "sonar.sources", "src/main"
  property "sonar.java.binaries", "build/tmp/kotlin-classes"
  property "sonar.tests", "src/test, ./src/androidTest/"
  property "sonar.java.coveragePlugin", "jacoco"
  // ver o caminho do xml
  property "sonar.coverage.jacoco.xmlReportPaths", "build/reports/jacoco/jacocoTestReportProd/jacocoTestReportProd.xml"
  // ver o caminho do xml
  property "sonar.junit.reportsPath", "build/reports/jacoco/jacocoTestReportProd/"
  property "sonar.android.lint.report", "build/reports/lint-results.xml"
  }
}
```

Depois adicione no build.gradle do seu módulo principal/base (normalmente chamado de :app), as seguintes linhas abaixo do apply dos outros plugins:

```groovy
apply plugin: "org.sonarqube"
apply from: './sonarqube.gradle'
```
Com essa configuração você pode rodar os cover de teste unitário e subir o projeto no SonarQube rodando:

```bash
./gradlew jacocoTestReport{nome da sua flavor} sonarqube
```

![enter image description here](https://raw.githubusercontent.com/GaldinoJr/TesteJacocoSonar/master/app/src/main/res/drawable-v24/Captura%20de%20Tela%202020-04-01%20%C3%A0s%2020.13.28.png)

Obs.: O comando muda conforme a flavor, rode o comando:

```bash
./gradlew tasks
```
Para obter o nome da task corretamente


Com essa configuração você pode subir o projeto no SonarQube rodando:
```bash
./gradlew sonarqube
```

# Importante
Para rodar localmente, igual está configurado, é preciso que tenha um server local rodando, se você quiser fazer isso, e estiver usando mac, pode utilizar o seguinte link:
https://mobiosolutions.com/install-sonarqube-installation-guide-mac-os/


# Conclusão
O cover fica um pouco diferente do jacoco e do sonar, acredito, que o sonar pega as duas colunas do jacoco missed instructions e missed branches e faz uma médias (missed instructions + missed branches) / 2.
Desculpem, foi o mais perto que consegui chegar de fazer um bom trabalho.