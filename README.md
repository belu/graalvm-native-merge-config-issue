# graalvm-native-merge-config-issue

Steps to reproduce:

1. Clean and package
```shell
mvn clean package
```

2. Run application and after a few seconds stop with pressing CTRL+C
```shell
java -agentlib:native-image-agent=config-merge-dir=config -jar target/*-runner.jar
```

3. Verify generated `reflect-config.json`
```shell
grep -3 "java.util.Optional" config/reflect-config.json 
```
Note that `allowUnsafeAccess` for field `value` is included.

4. Run application again, stop after a few seconds with pressing CTRL+C
```shell
java -agentlib:native-image-agent=config-merge-dir=config -jar target/*-runner.jar
```

5. Verify merged `reflect-config.json`
```shell
grep -3 "java.util.Optional" config/reflect-config.json
```
Note that `java.util.Optional` does no longer contain `allowUnsafeAccess` for field `value`.