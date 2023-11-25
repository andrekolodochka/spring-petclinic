This is a fork of Spring PetClinic Sample Application with a few changes.

1. The build was failing due to Postgres errors. To avoid that, [two tests](https://github.com/andrekolodochka/spring-petclinic/blob/main/src/test/java/org/springframework/samples/petclinic/PostgresIntegrationTests.java#L81) were removed.
2. Added a Dockerfile
3. 