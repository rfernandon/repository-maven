# Publicando um artifact em um repositório maven no Github

Este **README** fornece um guia passo a passo sobre como publicar um artifact Maven no GitHub usando o plugin _site-maven-plugin_.

Veja [GitHub Maven Plugins](https://github.com/github/maven-plugins/) para mais detalhes.

## 1.0 Configurando a auttenticação com o Github

Configure as informações de autenticação do github para que plugin _site-maven-plugin_ consiga fazer o push para o github.

Existem várias maneiras de configurar a autenticação do github, adicione uma das seguintes configurações ao seu **~/.m2/settings.xml**.

Usando seu nome de usuário e senha do github:

```xml
<settings>
  <servers>
    <server>
      <id>github</id>
      <username>GitHubLogin</username>
      <password>GitHubPassw0rd</password>
    </server>
  </servers>
</settings>
```

usando um token OAUTH2:

```xml
<settings>
  <servers>
    <server>
      <id>github</id>
      <password>OAUTH2TOKEN</password>
    </server>
  </servers>
</settings>
```

## 1.1 Publicando o artifact no epositório local

Antes de enviar o seu artifact ao repositório Maven no Github, precisamos publicá-lo em um repositório local.

Configure um repositório local no pom.xml do seu projeto:

```xml
<distributionManagement>
    <repository>
        <id>internal.repo</id>
        <name>Temporary Staging Repository</name>
        <url>file://${project.build.directory}/mvn-repo</url>
    </repository>
</distributionManagement>
````

Adicione agora a configuração do plugin **maven-deploy-plugin**:

```xml
<plugin>
    <artifactId>maven-deploy-plugin</artifactId>
    <version>2.8.2</version>
    <configuration>
        <altDeploymentRepository>internal.repo::default::file://${project.build.directory}/mvn-repo</altDeploymentRepository>
    </configuration>
</plugin>
```

Ao execute o comando `mvn clean deploy`, o plugin irá criar o repositório interno e publicar o artifact no diretório **target/mvn-repo/**.

## 1.2 Publicando o artifact no Github

Agora vamos precisar configurar o plugin _site-maven-plugin_ para que possamos fazer o upload do repositório local para o Github.

Informe o plugin sobre o novo servidor que você acabou de configurar, adicionando o seguinte ao seu pom.xml:

```xml
<properties>
  <!-- github server corresponds to entry in ~/.m2/settings.xml -->
  <github.global.server>github</github.global.server>
</properties>
```

Por fim, a última etapa é configurar o plugin **site-maven-plugin** para que seja enviado o repositório ao Github.

```xml
<plugin>
    <groupId>com.github.github</groupId>
    <artifactId>site-maven-plugin</artifactId>
    <version>0.11</version>
    <configuration>
        <message>Publishing to maven repository: ${project.artifactId}-${project.version}</message>
        <noJekyll>true</noJekyll>
        <outputDirectory>${project.build.directory}/mvn-repo</outputDirectory>
        <branch>refs/heads/main</branch>
        <includes>
            <include>**/*</include>
        </includes>
        <repositoryName>repository-maven</repositoryName>
        <repositoryOwner>rfernandon</repositoryOwner>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>site</goal>
            </goals>
            <phase>deploy</phase>
        </execution>
    </executions>
</plugin>
```

> Observe que esta solução substituirá seus artifact toda vez que você fizer uma nova implantação. Para desativar esse comportamento, defina **<merge>true</merge>** na configuração do _site-maven-plugin_.

Execute novamente o comando `mvn clean deploy` para fazer o upload de seu artifact para o github. A branch **main** será criada caso ela não existir.

Visite github.com em seu navegador, selecione o branch **main** e verifique se todos os arquivos binários foram carregados.

![REPO_001.png](/images/REPO_001.png)

Seu projeto maven agora está disponível para ser usado em outros projetos.

> Cada vez que for executado o comando `mvn clean deploy` em seu projeto, os artifact mais recentes serão enviados para o github.

## 2.0 Utilizando um artifact Maven publicado no Github

Para utilizar um artifact publicado no Github, precisamos informar em qual repositório esse artifact está:

```xml
<repositories>
    <repository>
        <id>rfernandon-repo</id>
        <url>https://raw.github.com/rfernandon/repository-maven/main/</url>
        <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
        </snapshots>
    </repository>
</repositories>
```

Agora precisamos adicionar a dependência:

```xml
<dependency>
    <groupId>YOUR.PROJECT.GROUPID</groupId>
    <artifactId>ARTIFACT-ID</artifactId>
    <version>VERSION</version>
</dependency>
```

Após configurar pom.xml, o projeto baixará automaticamente as dependências do repositório maven no Github.
