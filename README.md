# 🔒 Guia Completo: OWASP Dependency-Check para Desenvolvedores Java

## 📋 Índice
1. [O que é o Dependency-Check](#1-o-que-é-o-dependency-check)
2. [Instalação e Configuração](#2-instalação-e-configuração)
3. [Uso Local (Linha de Comando)](#3-uso-local-linha-de-comando)
4. [Integração com Maven](#4-integração-com-maven)
5. [Integração com Gradle](#5-integração-com-gradle)
6. [Integração com CI/CD](#6-integração-com-cicd)
7. [Análise de Relatórios](#7-análise-de-relatórios)
8. [Configurações Avançadas](#8-configurações-avançadas)
9. [Correção de Vulnerabilidades](#9-correção-de-vulnerabilidades)
10. [Boas Práticas](#10-boas-práticas)

---

## 1. O que é o Dependency-Check?

O **OWASP Dependency-Check** é uma ferramenta que identifica vulnerabilidades conhecidas (CVEs) nas dependências do seu projeto Java.

### 🎯 O que ele faz:
- ✅ Escaneia todas as dependências do projeto (diretas e transitivas)
- ✅ Compara com banco de dados de vulnerabilidades (NVD - National Vulnerability Database)
- ✅ Gera relatórios detalhados (HTML, JSON, XML, CSV)
- ✅ Pode quebrar o build se encontrar vulnerabilidades críticas
- ✅ Funciona com Maven, Gradle, Ant e linha de comando

### 📊 Exemplo de vulnerabilidade que ele detecta:
```
Log4Shell (CVE-2021-44228) - CRITICAL
Biblioteca: log4j-core 2.14.1
Severidade: 10.0 (Crítica)
Recomendação: Atualizar para versão 2.17.1 ou superior
```

---

## 2. Instalação e Configuração

### Opção A: Instalação via Linha de Comando (CLI)

#### **Windows:**
```powershell
# Baixar a versão mais recente
Invoke-WebRequest -Uri "https://github.com/jeremylong/DependencyCheck/releases/download/v9.0.7/dependency-check-9.0.7-release.zip" -OutFile "dependency-check.zip"

# Extrair
Expand-Archive -Path dependency-check.zip -DestinationPath C:\Tools\

# Adicionar ao PATH
$env:Path += ";C:\Tools\dependency-check\bin"

# Verificar instalação
dependency-check.bat --version
```

#### **Linux/Mac:**
```bash
# Baixar
wget https://github.com/jeremylong/DependencyCheck/releases/download/v9.0.7/dependency-check-9.0.7-release.zip

# Extrair
unzip dependency-check-9.0.7-release.zip -d /opt/

# Adicionar ao PATH (adicione ao ~/.bashrc ou ~/.zshrc)
export PATH=$PATH:/opt/dependency-check/bin

# Verificar instalação
dependency-check.sh --version
```

#### **Via Homebrew (Mac):**
```bash
brew install dependency-check
dependency-check --version
```

---

### Opção B: Não precisa instalar (usar via Maven/Gradle)

Se você usar Maven ou Gradle, não precisa instalar separadamente. Basta adicionar o plugin no seu projeto! 👇

---

## 3. Uso Local (Linha de Comando)

### 🚀 Primeiro Scan (Básico)

```bash
# Escanear projeto Maven
dependency-check.sh --project "MeuProjeto" --scan ./

# Escanear projeto Gradle
dependency-check.sh --project "MeuProjeto" --scan ./

# Especificar formato do relatório
dependency-check.sh --project "MeuProjeto" --scan ./ --format HTML

# Múltiplos formatos
dependency-check.sh --project "MeuProjeto" --scan ./ --format "HTML,JSON,XML"
```

### 📂 Estrutura de Comando Completo

```bash
dependency-check.sh \
  --project "Nome do Projeto" \
  --scan /caminho/do/projeto \
  --out /caminho/para/relatorio \
  --format HTML \
  --log /caminho/para/logs/dependency-check.log
```

### 🎯 Exemplos Práticos

#### **Exemplo 1: Scan Simples**
```bash
cd /meu-projeto-java
dependency-check.sh --project "API-Pagamentos" --scan ./
```

**Resultado:** Gera relatório `dependency-check-report.html` no diretório atual.

#### **Exemplo 2: Scan com Saída Customizada**
```bash
dependency-check.sh \
  --project "API-Pagamentos" \
  --scan ./ \
  --out ./reports/security \
  --format "HTML,JSON"
```

**Resultado:** Gera relatórios HTML e JSON em `./reports/security/`

#### **Exemplo 3: Scan com Limite de Severidade**
```bash
dependency-check.sh \
  --project "API-Pagamentos" \
  --scan ./ \
  --failOnCVSS 7 \
  --format HTML
```

**Resultado:** Falha se encontrar vulnerabilidades com CVSS >= 7

#### **Exemplo 4: Scan Completo com Todas as Opções**
```bash
dependency-check.sh \
  --project "API-Pagamentos" \
  --scan ./ \
  --out ./reports \
  --format "HTML,JSON,XML,CSV" \
  --log ./logs/dependency-check.log \
  --failOnCVSS 7 \
  --suppression ./dependency-check-suppressions.xml \
  --enableExperimental
```

---

## 4. Integração com Maven

### 📦 Passo 1: Adicionar Plugin no pom.xml

```xml
<project>
  <!-- ... outras configurações ... -->
  
  <build>
    <plugins>
      <!-- OWASP Dependency-Check Plugin -->
      <plugin>
        <groupId>org.owasp</groupId>
        <artifactId>dependency-check-maven</artifactId>
        <version>9.0.7</version>
        <configuration>
          <!-- Nome do projeto no relatório -->
          <name>API-Pagamentos</name>
          
          <!-- Formatos de saída -->
          <formats>
            <format>HTML</format>
            <format>JSON</format>
          </formats>
          
          <!-- Diretório de saída -->
          <outputDirectory>${project.build.directory}/dependency-check</outputDirectory>
          
          <!-- Falhar build se encontrar vulnerabilidades críticas -->
          <failBuildOnCVSS>7</failBuildOnCVSS>
          
          <!-- Suprimir falsos positivos (opcional) -->
          <suppressionFiles>
            <suppressionFile>dependency-check-suppressions.xml</suppressionFile>
          </suppressionFiles>
          
          <!-- Analisadores específicos -->
          <assemblyAnalyzerEnabled>true</assemblyAnalyzerEnabled>
          <jarAnalyzerEnabled>true</jarAnalyzerEnabled>
          <nodeAnalyzerEnabled>false</nodeAnalyzerEnabled>
          
          <!-- Cache do banco de dados NVD -->
          <dataDirectory>${project.build.directory}/dependency-check-data</dataDirectory>
          
          <!-- Atualização automática do banco de dados -->
          <autoUpdate>true</autoUpdate>
        </configuration>
        
        <executions>
          <execution>
            <goals>
              <goal>check</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

### 🚀 Passo 2: Executar o Scan

#### **Opção A: Apenas Gerar Relatório (não quebra build)**
```bash
mvn dependency-check:check
```

#### **Opção B: Verificar e Quebrar Build se Necessário**
```bash
mvn verify
```

#### **Opção C: Atualizar Banco de Dados Manualmente**
```bash
mvn dependency-check:update-only
```

#### **Opção D: Limpar Cache**
```bash
mvn dependency-check:purge
```

### 📊 Passo 3: Visualizar Relatório

Após executar, o relatório estará em:
```
target/dependency-check/dependency-check-report.html
```

Abra no navegador:
```bash
# Linux/Mac
open target/dependency-check/dependency-check-report.html

# Windows
start target/dependency-check/dependency-check-report.html
```

---

### 🎯 Configuração Recomendada para Projetos Reais

```xml
<plugin>
  <groupId>org.owasp</groupId>
  <artifactId>dependency-check-maven</artifactId>
  <version>9.0.7</version>
  <configuration>
    <name>${project.name}</name>
    
    <!-- Formatos múltiplos para diferentes usos -->
    <formats>
      <format>HTML</format>  <!-- Para visualização humana -->
      <format>JSON</format>  <!-- Para integração com outras ferramentas -->
      <format>JUNIT</format> <!-- Para CI/CD -->
    </formats>
    
    <!-- Falhar apenas em vulnerabilidades ALTAS e CRÍTICAS -->
    <failBuildOnCVSS>7</failBuildOnCVSS>
    
    <!-- Arquivo de supressões (falsos positivos) -->
    <suppressionFiles>
      <suppressionFile>${project.basedir}/dependency-check-suppressions.xml</suppressionFile>
    </suppressionFiles>
    
    <!-- Otimizações de performance -->
    <skipSystemScope>true</skipSystemScope>
    <skipTestScope>false</skipTestScope>
    
    <!-- Analisadores -->
    <assemblyAnalyzerEnabled>true</assemblyAnalyzerEnabled>
    <jarAnalyzerEnabled>true</jarAnalyzerEnabled>
    <nodeAnalyzerEnabled>false</nodeAnalyzerEnabled>
    <retireJsAnalyzerEnabled>false</retireJsAnalyzerEnabled>
    
    <!-- Cache centralizado (para builds mais rápidos) -->
    <dataDirectory>${user.home}/.m2/dependency-check-data</dataDirectory>
    
    <!-- Atualização automática -->
    <autoUpdate>true</autoUpdate>
    
    <!-- Timeout para downloads -->
    <connectionTimeout>30000</connectionTimeout>
  </configuration>
  
  <executions>
    <execution>
      <goals>
        <goal>check</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

---

### 📝 Passo 4: Criar Arquivo de Supressões (Opcional)

Crie o arquivo `dependency-check-suppressions.xml` na raiz do projeto:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<suppressions xmlns="https://jeremylong.github.io/DependencyCheck/dependency-suppression.1.3.xsd">
  
  <!-- Suprimir falso positivo específico -->
  <suppress>
    <notes>
      <![CDATA[
      Falso positivo: CVE-2023-12345 não afeta nossa versão
      Ticket: SECURITY-123
      ]]>
    </notes>
    <cve>CVE-2023-12345</cve>
  </suppress>
  
  <!-- Suprimir por arquivo específico -->
  <suppress>
    <notes>
      <![CDATA[
      Biblioteca interna sem vulnerabilidade real
      ]]>
    </notes>
    <filePath regex="true">.*internal-library-.*\.jar</filePath>
  </suppress>
  
  <!-- Suprimir por GAV (Group, Artifact, Version) -->
  <suppress>
    <notes>
      <![CDATA[
      Vulnerabilidade já corrigida em nossa versão customizada
      ]]>
    </notes>
    <gav regex="true">com\.example:custom-lib:.*</gav>
    <cve>CVE-2023-99999</cve>
  </suppress>
  
</suppressions>
```

---

### 🔄 Passo 5: Integrar com Lifecycle do Maven

Para executar automaticamente em fases específicas:

```xml
<plugin>
  <groupId>org.owasp</groupId>
  <artifactId>dependency-check-maven</artifactId>
  <version>9.0.7</version>
  <configuration>
    <!-- configurações aqui -->
  </configuration>
  
  <executions>
    <!-- Executar durante 'mvn verify' -->
    <execution>
      <id>check-dependencies</id>
      <phase>verify</phase>
      <goals>
        <goal>check</goal>
      </goals>
    </execution>
    
    <!-- Atualizar banco de dados durante 'mvn validate' -->
    <execution>
      <id>update-database</id>
      <phase>validate</phase>
      <goals>
        <goal>update-only</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

---

## 5. Integração com Gradle

### 📦 Passo 1: Adicionar Plugin no build.gradle

#### **Gradle Groovy DSL:**

```groovy
plugins {
    id 'java'
    id 'org.owasp.dependencycheck' version '9.0.7'
}

// Configuração do Dependency-Check
dependencyCheck {
    // Nome do projeto
    projectName = 'API-Pagamentos'
    
    // Formatos de saída
    formats = ['HTML', 'JSON', 'JUNIT']
    
    // Diretório de saída
    outputDirectory = file("${buildDir}/reports/dependency-check")
    
    // Falhar build em vulnerabilidades críticas
    failBuildOnCVSS = 7
    
    // Arquivo de supressões
    suppressionFile = file('dependency-check-suppressions.xml')
    
    // Analisadores
    analyzers {
        assemblyEnabled = true
        jarEnabled = true
        nodeEnabled = false
        retireJsEnabled = false
    }
    
    // Cache do banco de dados
    data {
        directory = file("${System.getProperty('user.home')}/.gradle/dependency-check-data")
    }
    
    // Atualização automática
    autoUpdate = true
    
    // Escopos a verificar
    skipConfigurations = ['testCompile', 'testImplementation']
}

// Executar automaticamente durante build
tasks.named('check') {
    dependsOn 'dependencyCheckAnalyze'
}
```

#### **Gradle Kotlin DSL (build.gradle.kts):**

```kotlin
plugins {
    java
    id("org.owasp.dependencycheck") version "9.0.7"
}

dependencyCheck {
    // Nome do projeto
    projectName.set("API-Pagamentos")
    
    // Formatos de saída
    formats = listOf("HTML", "JSON", "JUNIT")
    
    // Diretório de saída
    outputDirectory.set(file("${buildDir}/reports/dependency-check"))
    
    // Falhar build em vulnerabilidades críticas
    failBuildOnCVSS = 7.0f
    
    // Arquivo de supressões
    suppressionFile.set(file("dependency-check-suppressions.xml"))
    
    // Analisadores
    analyzers.apply {
        assemblyEnabled = true
        jarEnabled = true
        nodeEnabled = false
        retireJsEnabled = false
    }
    
    // Cache do banco de dados
    data.apply {
        directory.set(file("${System.getProperty("user.home")}/.gradle/dependency-check-data"))
    }
    
    // Atualização automática
    autoUpdate = true
}

tasks.named("check") {
    dependsOn("dependencyCheckAnalyze")
}
```

---

### 🚀 Passo 2: Executar o Scan

```bash
# Executar análise de dependências
./gradlew dependencyCheckAnalyze

# Executar com build completo
./gradlew build

# Apenas atualizar banco de dados
./gradlew dependencyCheckUpdate

# Limpar cache
./gradlew dependencyCheckPurge

# Ver todas as tasks disponíveis
./gradlew tasks --group="OWASP dependency-check"
```

---

### 📊 Passo 3: Visualizar Relatório

```bash
# Linux/Mac
open build/reports/dependency-check/dependency-check-report.html

# Windows
start build/reports/dependency-check/dependency-check-report.html
```

---

### 🎯 Configuração Avançada para Gradle

```groovy
dependencyCheck {
    projectName = 'API-Pagamentos'
    
    // Múltiplos formatos
    formats = ['HTML', 'JSON', 'XML', 'CSV', 'JUNIT', 'SARIF']
    
    // Diretórios
    outputDirectory = file("${buildDir}/reports/dependency-check")
    scanConfigurations = ['runtimeClasspath']
    skipConfigurations = ['testRuntimeClasspath']
    
    // Severidade
    failBuildOnCVSS = 7
    failOnError = true
    
    // Supressões
    suppressionFile = file('dependency-check-suppressions.xml')
    
    // Analisadores detalhados
    analyzers {
        assemblyEnabled = true
        jarEnabled = true
        archiveEnabled = true
        nodeEnabled = false
        nodeAuditEnabled = false
        retireJsEnabled = false
        nuspecEnabled = false
        nugetconfEnabled = false
        cocoapodsEnabled = false
        swiftEnabled = false
        bundleAuditEnabled = false
        pyDistributionEnabled = false
        pyPackageEnabled = false
        rubygemsEnabled = false
        cmakeEnabled = false
        autoconfEnabled = false
        composerEnabled = false
        golangDepEnabled = false
        golangModEnabled = false
    }
    
    // Configurações de rede
    connectionTimeout = 30000
    
    // Cache e performance
    data {
        directory = file("${System.getProperty('user.home')}/.gradle/dependency-check-data")
    }
    autoUpdate = true
    
    // NVD API Key (opcional, mas recomendado para evitar rate limiting)
    nvd {
        apiKey = System.getenv('NVD_API_KEY') ?: ''
        delay = 2000 // delay entre requisições em ms
    }
}
```

---

### 🔑 Passo 4: Obter NVD API Key (Recomendado)

A partir de 2023, o NVD implementou rate limiting. Para evitar lentidão:

1. **Obter API Key:**
   - Acesse: https://nvd.nist.gov/developers/request-an-api-key
   - Preencha o formulário
   - Receberá a key por email

2. **Configurar no Gradle:**

```groovy
dependencyCheck {
    nvd {
        apiKey = System.getenv('NVD_API_KEY')
    }
}
```

3. **Definir variável de ambiente:**

```bash
# Linux/Mac (~/.bashrc ou ~/.zshrc)
export NVD_API_KEY='sua-api-key-aqui'

# Windows (PowerShell)
$env:NVD_API_KEY='sua-api-key-aqui'

# Ou criar arquivo gradle.properties
echo "nvdApiKey=sua-api-key-aqui" >> ~/.gradle/gradle.properties
```

---

## 6. Integração com CI/CD

### 🔄 GitHub Actions

Crie `.github/workflows/dependency-check.yml`:

```yaml
name: OWASP Dependency Check

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]
  schedule:
    # Executar toda segunda-feira às 9h
    - cron: '0 9 * * 1'

jobs:
  dependency-check:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout código
        uses: actions/checkout@v4
      
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
          cache: 'maven'
      
      - name: Cache Dependency-Check Database
        uses: actions/cache@v3
        with:
          path: ~/.m2/dependency-check-data
          key: ${{ runner.os }}-dependency-check-${{ hashFiles('**/pom.xml') }}
          restore-keys: |
            ${{ runner.os }}-dependency-check-
      
      - name: Run Dependency-Check
        env:
          NVD_API_KEY: ${{ secrets.NVD_API_KEY }}
        run: |
          mvn dependency-check:check \
            -Dnvd.api.key=$NVD_API_KEY \
            -DfailBuildOnCVSS=7
      
      - name: Upload Relatório HTML
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: dependency-check-report
          path: target/dependency-check/dependency-check-report.html
      
      - name: Upload Relatório JSON
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: dependency-check-json
          path: target/dependency-check/dependency-check-report.json
      
      - name: Comentar PR com Resultados
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const report = JSON.parse(fs.readFileSync('target/dependency-check/dependency-check-report.json', 'utf8'));
            
            const vulnerabilities = report.dependencies
              .flatMap(dep => dep.vulnerabilities || [])
              .filter(vuln => vuln.cvssv3?.baseScore >= 7);
            
            if (vulnerabilities.length > 0) {
              const comment = `## ⚠️ Vulnerabilidades Encontradas\n\n` +
                `Foram encontradas ${vulnerabilities.length} vulnerabilidades críticas/altas:\n\n` +
                vulnerabilities.map(v => 
                  `- **${v.name}** (CVSS: ${v.cvssv3?.baseScore}) - ${v.description?.substring(0, 100)}...`
                ).join('\n');
              
              github.rest.issues.createComment({
                issue_number: context.issue.number,
                owner: context.repo.owner,
                repo: context.repo.repo,
                body: comment
              });
            }
```

---

### 🔄 GitLab CI

Crie `.gitlab-ci.yml`:

```yaml
stages:
  - security

dependency-check:
  stage: security
  image: maven:3.9-eclipse-temurin-17
  
  variables:
    MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"
  
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - .m2/repository
      - dependency-check-data
  
  before_script:
    - mkdir -p dependency-check-data
  
  script:
    - |
      mvn dependency-check:check \
        -Dnvd.api.key=$NVD_API_KEY \
        -DfailBuildOnCVSS=7 \
        -Ddata.directory=$CI_PROJECT_DIR/dependency-check-data
  
  artifacts:
    when: always
    paths:
      - target/dependency-check/dependency-check-report.html
      - target/dependency-check/dependency-check-report.json
    reports:
      junit: target/dependency-check/dependency-check-junit.xml
    expire_in: 30 days
  
  only:
    - main
    - develop
    - merge_requests
  
  allow_failure: false
```

---

### 🔄 Jenkins Pipeline

Crie `Jenkinsfile`:

```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9'
        jdk 'JDK 17'
    }
    
    environment {
        NVD_API_KEY = credentials('nvd-api-key')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Dependency Check') {
            steps {
                script {
                    sh """
                        mvn dependency-check:check \
                            -Dnvd.api.key=${NVD_API_KEY} \
                            -DfailBuildOnCVSS=7 \
                            -Dformats=HTML,JSON,JUNIT
                    """
                }
            }
        }
    }
    
    post {
        always {
            // Publicar relatório HTML
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'target/dependency-check',
                reportFiles: 'dependency-check-report.html',
                reportName: 'OWASP Dependency-Check Report'
            ])
            
            // Publicar resultados JUnit
            junit 'target/dependency-check/dependency-check-junit.xml'
            
            // Arquivar artefatos
            archiveArtifacts artifacts: 'target/dependency-check/**/*', allowEmptyArchive: true
        }
        
        failure {
            emailext(
                subject: "❌ Vulnerabilidades Encontradas - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <p>Foram encontradas vulnerabilidades críticas no projeto.</p>
                    <p>Verifique o relatório: ${env.BUILD_URL}OWASP_Dependency-Check_Report/</p>
                """,
                to: 'security-team@empresa.com',
                mimeType: 'text/html'
            )
        }
    }
}
```

---

### 🔄 Azure DevOps

Crie `azure-pipelines.yml`:

```yaml
trigger:
  branches:
    include:
      - main
      - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  MAVEN_CACHE_FOLDER: $(Pipeline.Workspace)/.m2/repository
  MAVEN_OPTS: '-Dmaven.repo.local=$(MAVEN_CACHE_FOLDER)'

steps:
  - task: Cache@2
    inputs:
      key: 'maven | "$(Agent.OS)" | **/pom.xml'
      restoreKeys: |
        maven | "$(Agent.OS)"
        maven
      path: $(MAVEN_CACHE_FOLDER)
    displayName: 'Cache Maven packages'
  
  - task: Cache@2
    inputs:
      key: 'dependency-check | "$(Agent.OS)"'
      path: $(Pipeline.Workspace)/dependency-check-data
    displayName: 'Cache Dependency-Check Database'
  
  - task: Maven@3
    inputs:
      mavenPomFile: 'pom.xml'
      goals: 'dependency-check:check'
      options: |
        -Dnvd.api.key=$(NVD_API_KEY)
        -DfailBuildOnCVSS=7
        -Ddata.directory=$(Pipeline.Workspace)/dependency-check-data
      publishJUnitResults: true
      testResultsFiles: '**/target/dependency-check/dependency-check-junit.xml'
      javaHomeOption: 'JDKVersion'
      jdkVersionOption: '1.17'
      mavenVersionOption: 'Default'
    displayName: 'Run OWASP Dependency-Check'
  
  - task: PublishBuildArtifacts@1
    condition: always()
    inputs:
      pathToPublish: 'target/dependency-check'
      artifactName: 'dependency-check-reports'
    displayName: 'Publish Dependency-Check Reports'
  
  - task: PublishTestResults@2
    condition: always()
    inputs:
      testResultsFormat: 'JUnit'
      testResultsFiles: '**/dependency-check-junit.xml'
      testRunTitle: 'OWASP Dependency-Check'
    displayName: 'Publish Test Results'
```

---

### 🔄 CircleCI

Crie `.circleci/config.yml`:

```yaml
version: 2.1

orbs:
  maven: circleci/maven@1.4

jobs:
  dependency-check:
    docker:
      - image: cimg/openjdk:17.0
    
    steps:
      - checkout
      
      - restore_cache:
          keys:
            - maven-deps-v1-{{ checksum "pom.xml" }}
            - maven-deps-v1-
      
      - restore_cache:
          keys:
            - dependency-check-db-v1-{{ epoch }}
            - dependency-check-db-v1-
      
      - run:
          name: Run Dependency-Check
          command: |
            mvn dependency-check:check \
              -Dnvd.api.key=$NVD_API_KEY \
              -DfailBuildOnCVSS=7
      
      - save_cache:
          key: maven-deps-v1-{{ checksum "pom.xml" }}
          paths:
            - ~/.m2
      
      - save_cache:
          key: dependency-check-db-v1-{{ epoch }}
          paths:
            - ~/.m2/dependency-check-data
      
      - store_artifacts:
          path: target/dependency-check
          destination: dependency-check-reports
      
      - store_test_results:
          path: target/dependency-check

workflows:
  version: 2
  build-and-security:
    jobs:
      - dependency-check:
          context: security-credentials
```

---

## 7. Análise de Relatórios

### 📊 Entendendo o Relatório HTML

O relatório HTML gerado contém várias seções:

#### **1. Summary (Resumo)**
```
Project: API-Pagamentos
Scan Date: 2025-11-19
Dependencies Scanned: 45
Vulnerabilities Found: 3
  - Critical: 1
  - High: 1
  - Medium: 1
  - Low: 0
```

#### **2. Dependency Details (Detalhes das Dependências)**

Para cada dependência vulnerável, você verá:

```
📦 log4j-core-2.14.1.jar
   Group: org.apache.logging.log4j
   Artifact: log4j-core
   Version: 2.14.1
   
   ⚠️ CVE-2021-44228 (Log4Shell)
   Severity: CRITICAL (CVSS: 10.0)
   Description: Apache Log4j2 permite execução remota de código...
   
   🔧 Recommendation:
   Atualizar para versão 2.17.1 ou superior
   
   🔗 References:
   - https://nvd.nist.gov/vuln/detail/CVE-2021-44228
   - https://logging.apache.org/log4j/2.x/security.html
```

---

### 📈 Entendendo CVSS Score

**CVSS (Common Vulnerability Scoring System)** é uma escala de 0 a 10:

| Score | Severidade | Ação Recomendada |
|-------|-----------|------------------|
| 9.0 - 10.0 | **CRITICAL** | ⚠️ Corrigir IMEDIATAMENTE |
| 7.0 - 8.9 | **HIGH** | 🔴 Corrigir em até 7 dias |
| 4.0 - 6.9 | **MEDIUM** | 🟡 Corrigir em até 30 dias |
| 0.1 - 3.9 | **LOW** | 🟢 Avaliar e planejar correção |

---

### 🔍 Analisando Relatório JSON

O relatório JSON é útil para automação:

```json
{
  "reportSchema": "1.1",
  "scanInfo": {
    "engineVersion": "9.0.7",
    "dataSource": [
      {
        "name": "NVD CVE Checked",
        "timestamp": "2025-11-19T10:00:00Z"
      }
    ]
  },
  "projectInfo": {
    "name": "API-Pagamentos",
    "reportDate": "2025-11-19T10:30:00Z",
    "credits": {
      "NVD": "https://nvd.nist.gov"
    }
  },
  "dependencies": [
    {
      "fileName": "log4j-core-2.14.1.jar",
      "filePath": "/path/to/log4j-core-2.14.1.jar",
      "md5": "abc123...",
      "sha1": "def456...",
      "sha256": "ghi789...",
      "packages": [
        {
          "id": "pkg:maven/org.apache.logging.log4j/log4j-core@2.14.1",
          "confidence": "HIGH",
          "url": "https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core/2.14.1"
        }
      ],
      "vulnerabilities": [
        {
          "source": "NVD",
          "name": "CVE-2021-44228",
          "severity": "CRITICAL",
          "cvssv3": {
            "baseScore": 10.0,
            "attackVector": "NETWORK",
            "attackComplexity": "LOW",
            "privilegesRequired": "NONE",
            "userInteraction": "NONE",
            "scope": "CHANGED",
            "confidentialityImpact": "HIGH",
            "integrityImpact": "HIGH",
            "availabilityImpact": "HIGH",
            "baseSeverity": "CRITICAL"
          },
          "description": "Apache Log4j2 2.0-beta9 through 2.15.0 (excluding security releases 2.12.2, 2.12.3, and 2.3.1) JNDI features used in configuration, log messages, and parameters do not protect against attacker controlled LDAP and other JNDI related endpoints. An attacker who can control log messages or log message parameters can execute arbitrary code loaded from LDAP servers when message lookup substitution is enabled.",
          "references": [
            {
              "source": "https://nvd.nist.gov/vuln/detail/CVE-2021-44228",
              "name": "CVE-2021-44228"
            }
          ]
        }
      ]
    }
  ]
}
```

---

### 🛠️ Script para Processar Relatório JSON

```bash
#!/bin/bash
# parse-dependency-check.sh

REPORT_FILE="target/dependency-check/dependency-check-report.json"

if [ ! -f "$REPORT_FILE" ]; then
    echo "❌ Relatório não encontrado: $REPORT_FILE"
    exit 1
fi

# Contar vulnerabilidades por severidade
CRITICAL=$(jq '[.dependencies[].vulnerabilities[]? | select(.severity == "CRITICAL")] | length' "$REPORT_FILE")
HIGH=$(jq '[.dependencies[].vulnerabilities[]? | select(.severity == "HIGH")] | length' "$REPORT_FILE")
MEDIUM=$(jq '[.dependencies[].vulnerabilities[]? | select(.severity == "MEDIUM")] | length' "$REPORT_FILE")
LOW=$(jq '[.dependencies[].vulnerabilities[]? | select(.severity == "LOW")] | length' "$REPORT_FILE")

echo "📊 Resumo de Vulnerabilidades:"
echo "   🔴 Critical: $CRITICAL"
echo "   🟠 High: $HIGH"
echo "   🟡 Medium: $MEDIUM"
echo "   🟢 Low: $LOW"
echo ""

# Listar vulnerabilidades críticas
if [ "$CRITICAL" -gt 0 ]; then
    echo "⚠️  VULNERABILIDADES CRÍTICAS:"
    jq -r '.dependencies[] | select(.vulnerabilities != null) | .vulnerabilities[] | select(.severity == "CRITICAL") | "   - \(.name) (CVSS: \(.cvssv3.baseScore)) em \(.source)"' "$REPORT_FILE"
    echo ""
fi

# Falhar se houver vulnerabilidades críticas ou altas
TOTAL_CRITICAL_HIGH=$((CRITICAL + HIGH))
if [ "$TOTAL_CRITICAL_HIGH" -gt 0 ]; then
    echo "❌ Build FALHOU: $TOTAL_CRITICAL_HIGH vulnerabilidades críticas/altas encontradas"
    exit 1
else
    echo "✅ Nenhuma vulnerabilidade crítica/alta encontrada"
    exit 0
fi
```

Uso:
```bash
chmod +x parse-dependency-check.sh
./parse-dependency-check.sh
```

---

## 8. Configurações Avançadas

### 🎯 Configuração de Analisadores

O Dependency-Check possui vários analisadores. Você pode habilitar/desabilitar conforme necessário:

#### **Maven:**
```xml
<configuration>
  <!-- Analisadores Java -->
  <assemblyAnalyzerEnabled>true</assemblyAnalyzerEnabled>
  <jarAnalyzerEnabled>true</jarAnalyzerEnabled>
  
  <!-- Analisadores JavaScript/Node -->
  <nodeAnalyzerEnabled>false</nodeAnalyzerEnabled>
  <nodeAuditAnalyzerEnabled>false</nodeAuditAnalyzerEnabled>
  <retireJsAnalyzerEnabled>false</retireJsAnalyzerEnabled>
  
  <!-- Analisadores .NET -->
  <nuspecAnalyzerEnabled>false</nuspecAnalyzerEnabled>
  <nugetconfAnalyzerEnabled>false</nugetconfAnalyzerEnabled>
  <assemblyAnalyzerEnabled>false</assemblyAnalyzerEnabled>
  
  <!-- Analisadores Python -->
  <pyDistributionAnalyzerEnabled>false</pyDistributionAnalyzerEnabled>
  <pyPackageAnalyzerEnabled>false</pyPackageAnalyzerEnabled>
  
  <!-- Analisadores Ruby -->
  <bundleAuditAnalyzerEnabled>false</bundleAuditAnalyzerEnabled>
  <rubygemsAnalyzerEnabled>false</rubygemsAnalyzerEnabled>
  
  <!-- Analisadores Go -->
  <golangDepEnabled>false</golangDepEnabled>
  <golangModEnabled>false</golangModEnabled>
  
  <!-- Outros -->
  <archiveAnalyzerEnabled>true</archiveAnalyzerEnabled>
  <centralAnalyzerEnabled>true</centralAnalyzerEnabled>
  <ossIndexAnalyzerEnabled>true</ossIndexAnalyzerEnabled>
</configuration>
```

---

### 🔧 Configuração de Proxy

Se sua empresa usa proxy:

#### **Maven:**
```xml
<configuration>
  <proxyServer>proxy.empresa.com</proxyServer>
  <proxyPort>8080</proxyPort>
  <proxyUsername>usuario</proxyUsername>
  <proxyPassword>senha</proxyPassword>
  <nonProxyHosts>localhost|127.0.0.1</nonProxyHosts>
</configuration>
```

#### **Gradle:**
```groovy
dependencyCheck {
    proxy {
        server = 'proxy.empresa.com'
        port = 8080
        username = 'usuario'
        password = 'senha'
        nonProxyHosts = ['localhost', '127.0.0.1']
    }
}
```

---

### 🗄️ Configuração de Database Mirroring

Para ambientes corporativos, você pode hospedar um mirror local do NVD:

#### **Maven:**
```xml
<configuration>
  <!-- Usar mirror local -->
  <cveUrl12Modified>https://nvd-mirror.empresa.com/nvdcve-1.1-modified.json.gz</cveUrl12Modified>
  <cveUrl20Modified>https://nvd-mirror.empresa.com/nvdcve-2.0-modified.json.gz</cveUrl20Modified>
  <cveUrl12Base>https://nvd-mirror.empresa.com/nvdcve-1.1-%d.json.gz</cveUrl12Base>
  <cveUrl20Base>https://nvd-mirror.empresa.com/nvdcve-2.0-%d.json.gz</cveUrl20Base>
</configuration>
```

---

### 📝 Supressões Avançadas

#### **Suprimir por CPE (Common Platform Enumeration):**
```xml
<suppress>
  <notes>
    <![CDATA[
    Falso positivo: Esta biblioteca não é afetada
    ]]>
  </notes>
  <cpe>cpe:/a:apache:commons-collections:3.2.1</cpe>
  <cve>CVE-2015-6420</cve>
</suppress>
```

#### **Suprimir por Package URL:**
```xml
<suppress>
  <notes>
    <![CDATA[
    Vulnerabilidade não aplicável ao nosso uso
    ]]>
  </notes>
  <packageUrl regex="true">^pkg:maven/com\.fasterxml\.jackson\.core/jackson\-databind@.*$</packageUrl>
  <vulnerabilityName>CVE-2020-36518</vulnerabilityName>
</suppress>
```

#### **Suprimir até data específica:**
```xml
<suppress until="2025-12-31Z">
  <notes>
    <![CDATA[
    Supressão temporária até correção ser disponibilizada
    Ticket: SEC-456
    ]]>
  </notes>
  <gav regex="true">com\.example:.*</gav>
  <cve>CVE-2024-12345</cve>
</suppress>
```

#### **Suprimir por CVSS Score:**
```xml
<suppress>
  <notes>
    <![CDATA[
    Aceitar vulnerabilidades baixas desta biblioteca
    ]]>
  </notes>
  <gav>com.example:internal-lib:1.0.0</gav>
  <cvssBelow>4.0</cvssBelow>
</suppress>
```

---

### 🔐 Integração com OSS Index

O Sonatype OSS Index é uma fonte adicional de vulnerabilidades:

#### **Maven:**
```xml
<configuration>
  <ossindexAnalyzerEnabled>true</ossindexAnalyzerEnabled>
  <ossindexAnalyzerUrl>https://ossindex.sonatype.org</ossindexAnalyzerUrl>
  
  <!-- Autenticação (opcional, mas recomendado) -->
  <ossindexAnalyzerUsername>${env.OSSINDEX_USER}</ossindexAnalyzerUsername>
  <ossindexAnalyzerPassword>${env.OSSINDEX_PASSWORD}</ossindexAnalyzerPassword>
</configuration>
```

Para obter credenciais: https://ossindex.sonatype.org/user/register

---

### 📊 Configuração de Relatórios Customizados

#### **Template HTML Customizado:**

```xml
<configuration>
  <format>HTML</format>
  <prettyPrint>true</prettyPrint>
  
  <!-- Template customizado -->
  <htmlReportTemplate>
    ${project.basedir}/custom-report-template.html
  </htmlReportTemplate>
</configuration>
```

---

### ⚡ Otimização de Performance

#### **1. Cache Centralizado:**

```xml
<!-- Maven -->
<configuration>
  <dataDirectory>${user.home}/.m2/dependency-check-data</dataDirectory>
</configuration>
```

```groovy
// Gradle
dependencyCheck {
    data {
        directory = file("${System.getProperty('user.home')}/.gradle/dependency-check-data")
    }
}
```

#### **2. Desabilitar Atualização Automática (CI):**

```xml
<configuration>
  <autoUpdate>false</autoUpdate>
</configuration>
```

Atualizar em job separado:
```bash
mvn dependency-check:update-only
```

#### **3. Skip em Builds Locais:**

```bash
# Maven
mvn clean install -Ddependency-check.skip=true

# Gradle
./gradlew build -x dependencyCheckAnalyze
```

#### **4. Análise Incremental:**

```xml
<configuration>
  <!-- Apenas verificar dependências modificadas -->
  <skipSystemScope>true</skipSystemScope>
  <skipTestScope>true</skipTestScope>
</configuration>
```

---

### 🎭 Profiles Maven para Diferentes Ambientes

```xml
<profiles>
  <!-- Profile para desenvolvimento local -->
  <profile>
    <id>dev</id>
    <activation>
      <activeByDefault>true</activeByDefault>
    </activation>
    <build>
      <plugins>
        <plugin>
          <groupId>org.owasp</groupId>
          <artifactId>dependency-check-maven</artifactId>
          <configuration>
            <skip>true</skip>
          </configuration>
        </plugin>
      </plugins>
    </build>
  </profile>
  
  <!-- Profile para CI/CD -->
  <profile>
    <id>ci</id>
    <build>
      <plugins>
        <plugin>
          <groupId>org.owasp</groupId>
          <artifactId>dependency-check-maven</artifactId>
          <configuration>
            <failBuildOnCVSS>7</failBuildOnCVSS>
            <autoUpdate>true</autoUpdate>
          </configuration>
        </plugin>
      </plugins>
    </build>
  </profile>
  
  <!-- Profile para auditoria de segurança completa -->
  <profile>
    <id>security-audit</id>
    <build>
      <plugins>
        <plugin>
          <groupId>org.owasp</groupId>
          <artifactId>dependency-check-maven</artifactId>
          <configuration>
            <failBuildOnCVSS>0</failBuildOnCVSS>
            <formats>
              <format>HTML</format>
              <format>JSON</format>
              <format>XML</format>
              <format>CSV</format>
            </formats>
          </configuration>
        </plugin>
      </plugins>
    </build>
  </profile>
</profiles>
```

Uso:
```bash
# Build normal (sem dependency-check)
mvn clean install

# Build CI
mvn clean verify -Pci

# Auditoria completa
mvn dependency-check:check -Psecurity-audit
```

---

## 9. Correção de Vulnerabilidades

### 🔧 Processo de Correção

#### **Passo 1: Identificar a Dependência Vulnerável**

No relatório, identifique:
- Nome da dependência
- Versão atual
- CVE encontrado
- Versão recomendada

#### **Passo 2: Verificar se é Dependência Direta ou Transitiva**

```bash
# Maven - Ver árvore de dependências
mvn dependency:tree -Dincludes=org.apache.logging.log4j:log4j-core

# Gradle - Ver árvore de dependências
./gradlew dependencies --configuration runtimeClasspath | grep log4j
```

#### **Passo 3: Atualizar Dependência**

##### **Se for Dependência Direta:**

**Maven (pom.xml):**
```xml
<dependencies>
  <!-- ANTES -->
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.14.1</version> <!-- Vulnerável -->
  </dependency>
  
  <!-- DEPOIS -->
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.17.1</version> <!-- Corrigido -->
  </dependency>
</dependencies>
```

**Gradle (build.gradle):**
```groovy
dependencies {
    // ANTES
    implementation 'org.apache.logging.log4j:log4j-core:2.14.1' // Vulnerável
    
    // DEPOIS
    implementation 'org.apache.logging.log4j:log4j-core:2.17.1' // Corrigido
}
```

##### **Se for Dependência Transitiva:**

**Opção 1: Atualizar Dependência Pai**

```bash
# Descobrir qual dependência traz a vulnerável
mvn dependency:tree | grep log4j-core
```

Resultado:
```
[INFO] +- com.example:some-library:jar:1.0.0:compile
[INFO]    \- org.apache.logging.log4j:log4j-core:jar:2.14.1:compile
```

Atualizar `some-library` para versão que use log4j seguro.

**Opção 2: Forçar Versão Específica (Maven)**

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
      <version>2.17.1</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

**Opção 3: Forçar Versão Específica (Gradle)**

```groovy
configurations.all {
    resolutionStrategy {
        force 'org.apache.logging.log4j:log4j-core:2.17.1'
    }
}
```

**Opção 4: Excluir e Adicionar Manualmente (Maven)**

```xml
<dependency>
  <groupId>com.example</groupId>
  <artifactId>some-library</artifactId>
  <version>1.0.0</version>
  <exclusions>
    <exclusion>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
    </exclusion>
  </exclusions>
</dependency>

<!-- Adicionar versão corrigida -->
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-core</artifactId>
  <version>2.17.1</version>
</dependency>
```

**Opção 5: Excluir e Adicionar Manualmente (Gradle)**

```groovy
dependencies {
    implementation('com.example:some-library:1.0.0') {
        exclude group: 'org.apache.logging.log4j', module: 'log4j-core'
    }
    
    // Adicionar versão corrigida
    implementation 'org.apache.logging.log4j:log4j-core:2.17.1'
}
```

---

### 🧪 Passo 4: Testar Após Atualização

```bash
# Limpar e rebuildar
mvn clean install

# Rodar testes
mvn test

# Verificar se vulnerabilidade foi corrigida
mvn dependency-check:check
```

---

### 📋 Passo 5: Documentar a Correção

Crie um commit descritivo:

```bash
git add pom.xml
git commit -m "fix(security): atualizar log4j-core para 2.17.1

Corrige CVE-2021-44228 (Log4Shell)
CVSS: 10.0 (Critical)

- Atualizado log4j-core de 2.14.1 para 2.17.1
- Testes executados com sucesso
- Dependency-check validado sem vulnerabilidades críticas

Refs: SEC-123"
```

---

### 🚫 Quando NÃO Há Versão Corrigida Disponível

#### **Opção 1: Aplicar Workaround**

Exemplo Log4Shell:
```java
// Desabilitar JNDI lookups
System.setProperty("log4j2.formatMsgNoLookups", "true");
```

#### **Opção 2: Substituir por Biblioteca Alternativa**

```xml
<!-- Remover biblioteca vulnerável -->
<dependency>
  <groupId>com.vulnerable</groupId>
  <artifactId>vulnerable-lib</artifactId>
  <version>1.0.0</version>
</dependency>

<!-- Adicionar alternativa segura -->
<dependency>
  <groupId>com.safe</groupId>
  <artifactId>safe-lib</artifactId>
  <version>2.0.0</version>
</dependency>
```

#### **Opção 3: Suprimir Temporariamente (com justificativa)**

```xml
<suppress until="2025-12-31Z">
  <notes>
    <![CDATA[
    Vulnerabilidade reconhecida mas sem correção disponível.
    Workaround aplicado: [descrever workaround]
    Mitigação: [descrever controles compensatórios]
    Ticket de acompanhamento: SEC-456
    Responsável: João Silva
    Revisão agendada: 2025-12-01
    ]]>
  </notes>
  <gav>com.vulnerable:vulnerable-lib:1.0.0</gav>
  <cve>CVE-2024-12345</cve>
</suppress>
```

---

### 📊 Exemplo Completo de Correção

**Cenário:** Encontrada vulnerabilidade crítica no Jackson Databind

**1. Relatório mostra:**
```
CVE-2020-36518 - CRITICAL (CVSS: 9.8)
Biblioteca: jackson-databind 2.9.8
Recomendação: Atualizar para 2.12.6.1 ou superior
```

**2. Verificar dependência:**
```bash
mvn dependency:tree | grep jackson-databind
```

**3. Resultado:**
```
[INFO] +- org.springframework.boot:spring-boot-starter-web:jar:2.3.0.RELEASE:compile
[INFO]    \- com.fasterxml.jackson.core:jackson-databind:jar:2.9.8:compile
```

**4. Solução - Atualizar Spring Boot:**

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <!-- ANTES: 2.3.0.RELEASE -->
  <version>2.6.7</version> <!-- DEPOIS: versão que inclui Jackson seguro -->
</parent>
```

**5. Verificar:**
```bash
mvn dependency:tree | grep jackson-databind
```

**6. Resultado após correção:**
```
[INFO] +- org.springframework.boot:spring-boot-starter-web:jar:2.6.7:compile
[INFO]    \- com.fasterxml.jackson.core:jackson-databind:jar:2.13.2.2:compile
```

**7. Validar:**
```bash
mvn clean test
mvn dependency-check:check
```

---

### 🔄 Automatizar Atualizações

#### **Maven Versions Plugin:**

```bash
# Verificar atualizações disponíveis
mvn versions:display-dependency-updates

# Atualizar para versões mais recentes
mvn versions:use-latest-releases

# Atualizar apenas patches de segurança
mvn versions:use-latest-versions -DallowSnapshots=false
```

#### **Dependabot (GitHub):**

Crie `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: "maven"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    
    # Agrupar atualizações de segurança
    groups:
      security-updates:
        patterns:
          - "*"
        update-types:
          - "patch"
          - "minor"
    
    # Labels automáticos
    labels:
      - "dependencies"
      - "security"
    
    # Reviewers automáticos
    reviewers:
      - "security-team"
    
    # Ignorar atualizações específicas
    ignore:
      - dependency-name: "com.example:stable-lib"
        update-types: ["version-update:semver-major"]
```

#### **Renovate Bot:**

Crie `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:base",
    ":dependencyDashboard",
    "schedule:weekly"
  ],
  "packageRules": [
    {
      "matchUpdateTypes": ["patch", "pin", "digest"],
      "automerge": true
    },
    {
      "matchPackagePatterns": ["*"],
      "matchUpdateTypes": ["minor", "patch"],
      "groupName": "all non-major dependencies",
      "groupSlug": "all-minor-patch"
    },
    {
      "matchDatasources": ["maven"],
      "addLabels": ["java", "dependencies"]
    }
  ],
  "vulnerabilityAlerts": {
    "enabled": true,
    "labels": ["security"],
    "assignees": ["@security-team"]
  }
}
```

---

## 10. Boas Práticas

### ✅ Checklist de Segurança de Dependências

#### **1. Configuração Inicial**
- [ ] Dependency-Check configurado no projeto
- [ ] Integrado ao CI/CD
- [ ] NVD API Key configurada
- [ ] Arquivo de supressões criado
- [ ] Threshold de CVSS definido (recomendado: 7.0)

#### **2. Processo de Build**
- [ ] Dependency-Check executa em todo PR
- [ ] Build falha em vulnerabilidades críticas/altas
- [ ] Relatórios são arquivados
- [ ] Notificações configuradas para falhas

#### **3. Monitoramento Contínuo**
- [ ] Scan semanal agendado
- [ ] Dashboard de vulnerabilidades atualizado
- [ ] Métricas de segurança acompanhadas
- [ ] SLA de correção definido

#### **4. Gestão de Vulnerabilidades**
- [ ] Processo de triagem definido
- [ ] Responsáveis designados
- [ ] Tickets de correção criados automaticamente
- [ ] Revisão periódica de supressões

---

### 🎯 Níveis de Maturidade

#### **Nível 1 - Básico**
```xml
<plugin>
  <groupId>org.owasp</groupId>
  <artifactId>dependency-check-maven</artifactId>
  <version>9.0.7</version>
  <executions>
    <execution>
      <goals>
        <goal>check</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```
- ✅ Scan manual ocasional
- ✅ Relatórios gerados localmente
- ⚠️ Sem automação

#### **Nível 2 - Intermediário**
```xml
<plugin>
  <groupId>org.owasp</groupId>
  <artifactId>dependency-check-maven</artifactId>
  <version>9.0.7</version>
  <configuration>
    <failBuildOnCVSS>7</failBuildOnCVSS>
    <formats>
      <format>HTML</format>
      <format>JSON</format>
    </formats>
  </configuration>
  <executions>
    <execution>
      <goals>
        <goal>check</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```
- ✅ Integrado ao CI/CD
- ✅ Build falha em vulnerabilidades críticas
- ✅ Relatórios arquivados
- ⚠️ Correções reativas

#### **Nível 3 - Avançado**
```xml
<plugin>
  <groupId>org.owasp</groupId>
  <artifactId>dependency-check-maven</artifactId>
  <version>9.0.7</version>
  <configuration>
    <failBuildOnCVSS>7</failBuildOnCVSS>
    <formats>
      <format>HTML</format>
      <format>JSON</format>
      <format>JUNIT</format>
      <format>SARIF</format>
    </formats>
    <suppressionFiles>
      <suppressionFile>dependency-check-suppressions.xml</suppressionFile>
    </suppressionFiles>
    <nvd>
      <apiKey>${env.NVD_API_KEY}</apiKey>
    </nvd>
    <ossindexAnalyzerEnabled>true</ossindexAnalyzerEnabled>
  </configuration>
  <executions>
    <execution>
      <goals>
        <goal>check</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```
- ✅ Scan automático em múltiplos ambientes
- ✅ Múltiplas fontes de vulnerabilidades (NVD + OSS Index)
- ✅ Supressões documentadas
- ✅ Métricas e dashboards
- ✅ Processo de correção definido

#### **Nível 4 - Expert**
- ✅ Tudo do nível 3, mais:
- ✅ Automação de atualizações (Dependabot/Renovate)
- ✅ Integração com Security Dashboard corporativo
- ✅ Políticas de segurança automatizadas
- ✅ SLA de correção por severidade
- ✅ Treinamento contínuo do time
- ✅ Contribuição para comunidade (reportar falsos positivos)

---

### 📊 Métricas Recomendadas

#### **KPIs de Segurança de Dependências:**

1. **Mean Time to Remediate (MTTR)**
   - Crítico: < 24 horas
   - Alto: < 7 dias
   - Médio: < 30 dias
   - Baixo: < 90 dias

2. **Vulnerabilities by Severity**
   ```
   Crítico: 0 (meta)
   Alto: < 5
   Médio: < 20
   Baixo: < 50
   ```

3. **Dependency Freshness**
   - % de dependências atualizadas nos últimos 6 meses: > 80%
   - % de dependências com 2+ anos: < 10%

4. **Coverage**
   - % de projetos com Dependency-Check: 100%
   - % de builds com scan automático: 100%

---

### 🚨 Alertas e Notificações

#### **Slack Integration:**

```bash
#!/bin/bash
# notify-slack.sh

WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
REPORT_FILE="target/dependency-check/dependency-check-report.json"

CRITICAL=$(jq '[.dependencies[].vulnerabilities[]? | select(.severity == "CRITICAL")] | length' "$REPORT_FILE")
HIGH=$(jq '[.dependencies[].vulnerabilities[]? | select(.severity == "HIGH")] | length' "$REPORT_FILE")

if [ "$CRITICAL" -gt 0 ] || [ "$HIGH" -gt 0 ]; then
  MESSAGE="🚨 *Vulnerabilidades Encontradas!*\n\n"
  MESSAGE+="🔴 Críticas: $CRITICAL\n"
  MESSAGE+="🟠 Altas: $HIGH\n\n"
  MESSAGE+="Projeto: ${PROJECT_NAME}\n"
  MESSAGE+="Branch: ${GIT_BRANCH}\n"
  MESSAGE+="<${BUILD_URL}|Ver Relatório>"
  
  curl -X POST "$WEBHOOK_URL" \
    -H 'Content-Type: application/json' \
    -d "{\"text\": \"$MESSAGE\"}"
fi
```

#### **Email Integration (Maven):**

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-antrun-plugin</artifactId>
  <version>3.1.0</version>
  <executions>
    <execution>
      <phase>verify</phase>
      <goals>
        <goal>run</goal>
      </goals>
      <configuration>
        <target>
          <mail from="ci@empresa.com"
                tolist="security-team@empresa.com"
                subject="[SECURITY] Vulnerabilidades Encontradas - ${project.name}"
                messagefile="target/dependency-check/dependency-check-report.html"
                messagemimetype="text/html"
                mailhost="smtp.empresa.com"
                mailport="587"
                user="ci@empresa.com"
                password="${env.SMTP_PASSWORD}"/>
        </target>
      </configuration>
    </execution>
  </executions>
</plugin>
```

---

### 🔍 Troubleshooting

#### **Problema 1: Build muito lento**

**Causa:** Download do banco de dados NVD

**Solução:**
```xml
<configuration>
  <!-- Usar cache centralizado -->
  <dataDirectory>${user.home}/.m2/dependency-check-data</dataDirectory>
  
  <!-- Desabilitar atualização automática em builds locais -->
  <autoUpdate>false</autoUpdate>
  
  <!-- Usar NVD API Key -->
  <nvd>
    <apiKey>${env.NVD_API_KEY}</apiKey>
  </nvd>
</configuration>
```

#### **Problema 2: Rate Limiting do NVD**

**Erro:**
```
[ERROR] Failed to download NVD data: 403 Forbidden
```

**Solução:**
1. Obter API Key: https://nvd.nist.gov/developers/request-an-api-key
2. Configurar:
```bash
export NVD_API_KEY='sua-key-aqui'
```

#### **Problema 3: Falsos Positivos**

**Sintoma:** CVE reportado não afeta sua versão

**Solução:**
```xml
<!-- dependency-check-suppressions.xml -->
<suppress>
  <notes>
    <![CDATA[
    CVE-2023-12345 não afeta versão 1.2.3 conforme:
    https://github.com/project/security/advisories/GHSA-xxxx
    ]]>
  </notes>
  <gav>com.example:library:1.2.3</gav>
  <cve>CVE-2023-12345</cve>
</suppress>
```

#### **Problema 4: Memória Insuficiente**

**Erro:**
```
[ERROR] Java heap space
```

**Solução Maven:**
```bash
export MAVEN_OPTS="-Xmx2048m"
mvn dependency-check:check
```

**Solução Gradle:**
```groovy
// gradle.properties
org.gradle.jvmargs=-Xmx2048m
```

#### **Problema 5: Proxy Corporativo**

**Erro:**
```
[ERROR] Connection timeout
```

**Solução:**
```xml
<configuration>
  <proxyServer>proxy.empresa.com</proxyServer>
  <proxyPort>8080</proxyPort>
  <proxyUsername>${env.PROXY_USER}</proxyUsername>
  <proxyPassword>${env.PROXY_PASS}</proxyPassword>
</configuration>
```

---

### 📚 Recursos Adicionais

#### **Documentação Oficial:**
- 🔗 [OWASP Dependency-Check](https://jeremylong.github.io/DependencyCheck/)
- 🔗 [Maven Plugin](https://jeremylong.github.io/DependencyCheck/dependency-check-maven/)
- 🔗 [Gradle Plugin](https://jeremylong.github.io/DependencyCheck/dependency-check-gradle/)

#### **Bases de Dados de Vulnerabilidades:**
- 🔗 [NVD - National Vulnerability Database](https://nvd.nist.gov/)
- 🔗 [OSS Index](https://ossindex.sonatype.org/)
- 🔗 [GitHub Advisory Database](https://github.com/advisories)
- 🔗 [Snyk Vulnerability DB](https://security.snyk.io/)

#### **Ferramentas Complementares:**
- 🔗 [Snyk](https://snyk.io/) - Scan de vulnerabilidades + correção automatizada
- 🔗 [Dependabot](https://github.com/dependabot) - Atualizações automáticas
- 🔗 [Renovate](https://www.mend.io/free-developer-tools/renovate/) - Gestão de dependências
- 🔗 [OWASP Dependency-Track](https://dependencytrack.org/) - Dashboard centralizado

#### **Padrões e Frameworks:**
- 🔗 [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- 🔗 [CWE - Common Weakness Enumeration](https://cwe.mitre.org/)
- 🔗 [CVSS Calculator](https://www.first.org/cvss/calculator/3.1)

---

### 🎓 Treinamento do Time

#### **Workshop Sugerido (2 horas):**

**Parte 1: Teoria (30 min)**
- O que são vulnerabilidades de dependências?
- Como funcionam CVEs e CVSS?
- Impacto de vulnerabilidades conhecidas (casos reais)
- Responsabilidade compartilhada

**Parte 2: Hands-On (60 min)**
- Instalar e configurar Dependency-Check
- Rodar primeiro scan
- Analisar relatório
- Corrigir vulnerabilidade real
- Criar supressão documentada

**Parte 3: Integração (30 min)**
- Configurar no CI/CD
- Definir políticas de segurança
- Processo de triagem e correção
- Q&A

#### **Materiais de Apoio:**
```markdown
# Cheat Sheet - Dependency-Check

## Comandos Rápidos

### Maven
mvn dependency-check:check              # Rodar scan
mvn dependency-check:update-only        # Atualizar DB
mvn dependency-check:purge              # Limpar cache

### Gradle
./gradlew dependencyCheckAnalyze        # Rodar scan
./gradlew dependencyCheckUpdate         # Atualizar DB
./gradlew dependencyCheckPurge          # Limpar cache

## Configuração Mínima (Maven)
<plugin>
  <groupId>org.owasp</groupId>
  <artifactId>dependency-check-maven</artifactId>
  <version>9.0.7</version>
  <configuration>
    <failBuildOnCVSS>7</failBuildOnCVSS>
  </configuration>
</plugin>

## Severidade CVSS
9.0-10.0  = CRITICAL (Corrigir AGORA)
7.0-8.9   = HIGH     (Corrigir em 7 dias)
4.0-6.9   = MEDIUM   (Corrigir em 30 dias)
0.1-3.9   = LOW      (Avaliar e planejar)

## Obter NVD API Key
https://nvd.nist.gov/developers/request-an-api-key

## Suprimir Falso Positivo
<suppress>
  <notes>Justificativa aqui</notes>
  <gav>group:artifact:version</gav>
  <cve>CVE-2023-12345</cve>
</suppress>
```

### 🎯 Próximos Passos

Agora que você tem o guia completo, siga esta ordem:

#### **Semana 1: Setup Básico**
- [ ] Instalar Dependency-Check localmente
- [ ] Adicionar plugin no projeto
- [ ] Rodar primeiro scan
- [ ] Analisar relatório

#### **Semana 2: Correções**
- [ ] Identificar vulnerabilidades críticas/altas
- [ ] Atualizar dependências
- [ ] Criar arquivo de supressões
- [ ] Validar correções

#### **Semana 3: Automação**
- [ ] Integrar com CI/CD
- [ ] Configurar notificações
- [ ] Definir políticas de build
- [ ] Documentar processo

#### **Semana 4: Melhoria Contínua**
- [ ] Configurar Dependabot/Renovate
- [ ] Criar dashboard de métricas
- [ ] Treinar time
- [ ] Revisar processo

---

### 💡 Dicas Finais

1. **Comece Simples:** Não tente implementar tudo de uma vez
2. **Priorize Críticos:** Foque primeiro em vulnerabilidades críticas/altas
3. **Documente Tudo:** Especialmente supressões e decisões
4. **Automatize:** Quanto mais automático, melhor
5. **Eduque o Time:** Segurança é responsabilidade de todos
6. **Monitore Continuamente:** Novas vulnerabilidades surgem diariamente
7. **Seja Pragmático:** Nem toda vulnerabilidade precisa ser corrigida imediatamente

---

### 🤝 Contribuindo

Encontrou algum erro ou tem sugestões? 

- 📧 Email: devmasterteam@exemplo.com
- 🐛 Issues: github.com/seu-repo/issues
- 💬 Discussões: github.com/seu-repo/discussions

---

### 📄 Licença

Este guia é disponibilizado sob licença MIT. Sinta-se livre para usar, modificar e compartilhar!

---

### ✅ Checklist Final

Antes de considerar seu projeto "seguro":

- [ ] Dependency-Check configurado e rodando
- [ ] Build falha em vulnerabilidades críticas/altas
- [ ] Processo de correção documentado
- [ ] Time treinado
- [ ] Métricas sendo acompanhadas
- [ ] Automação de atualizações configurada
- [ ] Revisão periódica agendada
- [ ] Documentação atualizada

---

## 🎉 Conclusão

**Lembre-se:**
> "Segurança não é um produto, é um processo." - Bruce Schneier

A implementação do Dependency-Check é apenas o primeiro passo. O verdadeiro valor vem do processo contínuo de:
- ✅ Monitoramento
- ✅ Correção
- ✅ Aprendizado
- ✅ Melhoria

**Boa sorte e bons scans! 🚀🔒**

---

**Última atualização:** 2025-11-19  
**Versão do Guia:** 1.0  
**Versão do Dependency-Check:** 9.0.7
