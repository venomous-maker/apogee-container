# apogee-container

Dependency injection for [Apogee](https://github.com/venomous-maker): bind
what a type means once, then ask for it anywhere. Classes are built by wiring
their constructors' parameters, interfaces included.

```grav
import "std"
import "apogee-container"

let c = new Container()
c.singleton::<Mailer>(fn(c: Container) -> Mailer { return new SmtpMailer(Config.getString("mail.host")) })
c.bindClass::<UserRepository, SqlUserRepository>()   // an interface to a class, autowired
c.scoped::<RequestContext>(fn(c: Container) -> RequestContext { return new RequestContext() })

let users = c.make::<UserService>()                  // constructor(repo: UserRepository, mail: Mailer)
let request = c.scope()                              // one per request: its own scoped values
```

Keys are types, by their fully-qualified names (`nameof(T)`), so an interface
is a key as well as a class. Plain-name keys exist as well (`bindNamed`,
`makeNamed`, `alias`).

## Lifetimes

| | |
|---|---|
| `bind::<T>(f)` / `bindClass::<T, C>()` | transient: a new value each time |
| `singleton::<T>(f)` / `singletonClass::<T, C>()` / `registerSingleton::<C>()` | one for the container and every scope of it, made on first use, exactly once even with many threads asking at once |
| `scoped::<T>(f)` / `scopedClass::<T, C>()` | one per `scope()` (a request, a job); resolving one from the root is an error |
| `instance::<T>(value)` | that value |

## Autowiring

A class with reflection is built by the container without a binding: a class
named by `register::<C>()`, or any class in a hierarchy with a decorator
(`@Service class UserService { ... }`). Of its constructors, the one with the
most parameters that can all be resolved is used. Parameters of interface or
class type are resolved; numbers, strings, functions and nullable values
cannot be (bind such a class with a factory). The plan is worked out once and
remembered, so the next build does no reflection lookups.

## More than a container needs

- **Contextual bindings:** `c.when::<ReportJob, Clock>(f)` gives `ReportJob`
  its own `Clock`; every other class keeps the ordinary one.
- **Extenders:** `c.extend::<Cache>(fn(cache: Cache, c: Container) -> Cache { return new LoggingCache(cache) })`
  wraps or replaces every `Cache` made.
- **Resolving hooks:** `c.resolving::<Model>(f)` sees each one as it is made.
- **Tags:** `c.tag::<CsvExporter>("exporters")`, then `c.tagged("exporters")`.
- **Method injection:** `c.call(controller, "show", [routeId])` calls a
  method with its class-typed parameters resolved and the others taken in
  order from the list (how controllers get both services and route values).
- **Aliases**, `has`/`bound`, `forget`, `resetSingletons`.

## Errors

`ContainerError` says what could not be resolved and what needed it:
`nothing is bound to app.Mailer (needed by app.SignupController ->
app.UserService)`, `dependency cycle: A -> B -> A`, `cannot build Port: no
constructor's parameters can all be resolved, tried (int)`.

## Speed

`--release`, one thread: an already-made singleton resolves in about 90 ns
(no lock: its value is published behind an atomic flag), a `scope()` costs
about 240 ns (it allocates nothing until it holds a scoped value), and
autowiring a class with two dependencies, objects made and freed included,
about 1.1 µs.

## Tests

```bash
grav test tests/container.grav
```

Needs a grav with `nameof`, `ClassInfo.forName` and reflective calls that
take interface arguments (all added for this package).

MIT licensed.
