# OpenRemote Groovy Sandbox

This repository is an OpenRemote-maintained fork of
[`craftercms/groovy-sandbox`](https://github.com/craftercms/groovy-sandbox),
which itself derives from the original
[`jenkinsci/groovy-sandbox`](https://github.com/jenkinsci/groovy-sandbox)
project.

The library provides a Groovy compile-time transformer that rewrites script
operations so they can be intercepted by application code. OpenRemote uses this
as part of its Groovy rules hardening work.

The Java package name remains `org.kohsuke.groovy.sandbox` for compatibility
with existing consumers, including OpenRemote code that imports classes such as
`org.kohsuke.groovy.sandbox.SandboxTransformer`.

## Fork Status

Current baseline:

- Forked from `craftercms/groovy-sandbox` `develop` at commit `97360e2`.
- Maven coordinates changed to `org.openremote:groovy-sandbox`.
- Java package names kept unchanged for API compatibility.
- Java 21 and Groovy 5 are the current build targets.

Relevant security and correctness fixes from `jenkinsci/groovy-sandbox` should
be documented here as they are manually ported into this fork. This fork should
not blindly cherry-pick Jenkins build, release, plugin, or CI metadata.

## Maven Coordinates

The Maven coordinates for this fork are:

```xml
<dependency>
    <groupId>org.openremote</groupId>
    <artifactId>groovy-sandbox</artifactId>
    <version>1.27.6-SNAPSHOT</version>
</dependency>
```

Publishing targets are intentionally not configured in this repository yet.
Snapshots should be published only after OpenRemote has validated the fork
against its own rules tests.

## Build

Requirements:

- Java 21
- Maven 3.9 or newer

Run the test suite:

```sh
mvn -B test
```

Build the jar, test jar, and sources jar:

```sh
mvn -B package
```

The current POM pins Groovy to `5.0.6`.

## Basic Usage

Add `SandboxTransformer` to a `CompilerConfiguration` before compiling the
Groovy script:

```groovy
def cc = new CompilerConfiguration()
cc.addCompilationCustomizers(new SandboxTransformer())

def shell = new GroovyShell(new Binding(), cc)
```

Register an interceptor around script execution:

```groovy
def sandbox = new GroovyValueFilter() {
    Object filter(Object value) {
        throw new SecurityException("Denied")
    }
}

sandbox.register()
try {
    shell.evaluate("println 'hello'")
} finally {
    sandbox.unregister()
}
```

See `src/test/groovy/org/kohsuke/groovy/sandbox/robot` for an allowlist-style
example that permits selected application objects and rejects everything else.

## Security Model

This library is defense in depth. It is not a complete JVM security boundary by
itself.

Important constraints:

- Interceptors are thread-specific. Scripts must not be allowed to create or
  control threads or executor services that run outside the registered
  interceptor context.
- Use an allowlist model. Blocking a known-dangerous method is not enough
  because another allowed method may call it internally.
- Reflection, class loading, process execution, filesystem access, networking,
  and Groovy dynamic hooks need explicit policy decisions.
- OpenRemote should combine this transformer with compiler restrictions,
  narrow rule DSL facades, and runtime containment before enabling untrusted
  Groovy authoring.

For OpenRemote, this means non-superuser Groovy rule editing should remain
disabled until this fork has a tested allowlist and OpenRemote has additional
runtime containment.

## License And Attribution

This OpenRemote fork distribution is licensed under the GNU Affero General
Public License version 3 or later. See `LICENSE.md`.

The upstream-origin code remains covered by the MIT License. The MIT license
text and original upstream copyright notice are preserved in `LICENSE-MIT.md`.
See `NOTICE.md` for fork attribution and OpenRemote contribution licensing
notes.

OpenRemote-specific changes should be documented in this repository without
removing existing Kohsuke Kawaguchi, CloudBees, Jenkins, CrafterCMS, or other
upstream contributor attribution.
