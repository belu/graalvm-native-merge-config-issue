# graalvm-native-merge-config-issue

Steps to reproduce:

1. Clean and package

```shell
mvn clean package
```

2. Run application and after a few seconds stop with pressing CTRL+C

```shell
java -agentlib:native-image-agent=config-merge-dir=src/main/resources/META-INF/native-image -jar target/*-runner.jar
```

3. Verify generated `reflect-config.json`

```shell
grep -3 "java.util.Optional" src/main/resources/META-INF/native-image/reflect-config.json 
```

Note that `allowUnsafeAccess` for field `value` is included.

4. Run application again, stop after a few seconds with pressing CTRL+C

```shell
java -agentlib:native-image-agent=config-merge-dir=src/main/resources/META-INF/native-image -jar target/*-runner.jar
```

5. Verify merged `reflect-config.json`

```shell
grep -3 "java.util.Optional" src/main/resources/META-INF/native-image/reflect-config.json
```

Note that `java.util.Optional` does no longer contain `allowUnsafeAccess` for field `value`.

6. Build native image

```shell
mvn package -Pnative
```

7. Run native executable

```shell
./target/microquark-1.0.0-SNAPSHOT-runner
```

The application crashes with the following error:

```shell
com.oracle.svm.core.jdk.UnsupportedFeatureError: The offset of private final
java.lang.Object java.util.Optional.value is accessed without the field being first
registered as unsafe accessed. Please register the field as unsafe accessed. You can do so
with a reflection configuration that contains an entry for the field with the attribute
"allowUnsafeAccess": true. Such a configuration file can be generated for you.
Read BuildConfiguration.md and Reflection.md for details.
```

8. Now edit `reflect-config.json` manually and change the entry of `java.util.Optional` to include
   the `allowUnsafeAccess=true`

```shell
{
"name":"java.util.Optional",
"allDeclaredFields":true,
"allDeclaredMethods":true,
"fields":[
{"name":"value", "allowUnsafeAccess":true}
]
},
```

9. Build the native image again

```shell
mvn package -Pnative
```

10. The application runs without crashing

```shell
./target/microquark-1.0.0-SNAPSHOT-runner
```
