# scala-nimbus-jose-jwt

[ ![Download](https://api.bintray.com/packages/guizmaii/maven/scala-nimbus-jose-jwt/images/download.svg) ](https://bintray.com/guizmaii/maven/scala-nimbus-jose-jwt/_latestVersion)
[![Build Status](https://travis-ci.org/guizmaii/scala-nimbus-jose-jwt.svg?branch=master)](https://travis-ci.org/guizmaii/scala-nimbus-jose-jwt)
[![codecov](https://codecov.io/gh/guizmaii/scala-nimbus-jose-jwt/branch/master/graph/badge.svg)](https://codecov.io/gh/guizmaii/scala-nimbus-jose-jwt)

**Small, simple and opinionated JWT token validator for Scala.**

## Goal

**Provide a very simple API to help people do JWT token validation.**

This project uses Nimbus JOSE + JWT (https://connect2id.com/products/nimbus-jose-jwt) to validate JWT tokens.
The aim of this project is not, and will never be, to provide a Scala interface to Nimbus JOSE + JWT.

I chose Nimbus JOSE + JWT because it seems to be audited and battle-tested.

The code size is smalll in order to be as readable as possible, so as free of bugs as possible.

## Setup

`libraryDependencies += "com.guizmaii" %% "scala-nimbus-jose-jwt" % "0.4.0"`

## Use

The API is very simple:

```
final case class JwtToken(content: String) extends AnyVal

trait JwtValidator {
  def validate(jwtToken: JwtToken): Either[BadJWTException, (JwtToken, JWTClaimsSet)]
}
```

### Available `JwtValidator` implementations

#### 1. ConfigurableJwtValidator

The more flexible implementation of the `JwtValidator` interface.

It only requires a `JWKSource` instance.    
For more information on the different `JWKSource` implementations Nimbus provides, look at the classes in the `com.nimbusds.jose.jwk.source` package here: https://www.javadoc.io/doc/com.nimbusds/nimbus-jose-jwt

Example of use:
```scala
import java.net.URL

import com.guizmaii.scalajwt.JwtToken
import com.guizmaii.scalajwt.ConfigurableJwtValidator
import com.nimbusds.jose.jwk.source.{JWKSource, RemoteJWKSet}
import com.nimbusds.jose.proc.SecurityContext
import com.nimbusds.jwt.JWTClaimsSet
import com.nimbusds.jwt.proc.BadJWTException

val token: JwtToken = JwtToken(content = "...")

val jwkSet: JWKSource[SecurityContext] = new RemoteJWKSet(
    new URL(s"https://your.jwks.prodvider.example.com/.well-known/jwks.json"))
    
val result: Either[BadJWTException, (JwtToken, JWTClaimsSet)] = ConfigurableJwtValidator(jwkSet).validate(token)
```

For more information on JWKs, you could read:   
  - Auth0 doc: https://auth0.com/docs/jwks    
  - Nimbus doc: https://connect2id.com/products/server/docs/api/jwk-set       
  - AWS Cognito doc: https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-with-identity-providers.html#amazon-cognito-identity-user-pools-using-id-and-access-tokens-in-web-api

Other constructor parameters are:

  - `maybeCtx: Option[SecurityContext] = None`   
    (Optional) Security context.    
    Default is `null` (no Security Context).
    
  - `maybeJwsAlgorithm: Option[JWSAlgorithm] = None`   
    (Optional) JWSAlgorithm.   
    Default is `RS256`.
    
  - `additionalChecks: List[(JWTClaimsSet, SecurityContext) => Option[BadJWTException]] = List.empty`   
    (Optional) List of additional checks that will be executed on the JWT token passed.    
    Default is an empty List.
    
    Some "additional checks" are already implemented in the object `ProvidedAdditionalChelcks`.

#### 2. AwsCognitoJwtValidator

Example of use:
```scala
import com.guizmaii.scalajwt.{AwsCognitoJwtValidator, CognitoUserPoolId, JwtToken, S3Region}

val jwtToken = JwtToken(content = "...")
val s3Region = S3Region(value = "eu-west-1")
val cognitoUserPoolId = CognitoUserPoolId(value = "...")

val awsCognitoJwtValidator = AwsCognitoJwtValidator(s3Region, cognitoUserPoolId).validate(jwtToken)
```
