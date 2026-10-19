# mlp-proto

The wire schemas of the **Media Lane Protocol (MLP)** — the class bodies a peer
may need to decode.

The protocol has two layers, and only one of them is here:

- the **fragment framing** — the header, the TLV registry, the object
  lifecycle — is a byte-level grammar that is not protobuf. It is specified in
  the [specification](../mlp) and implemented in
  [`mlp-core`](../mlp-core) (`media::pdu`, `media::object`);
- the **class bodies** — the control, cursor, feedback and path messages — are
  protobuf, and they live here.

`class = MEDIA` is deliberately *not* in this file. The protocol does not
interpret an update's bytes; a codec does, and a codec brings its own schema.

## Layout

```
proto/mlp/v1/mlp.proto     the normative schema
src/generated/mlp.ts       the generated TypeScript bindings
tools/generate.mjs         the generator
```

The Rust bindings are generated in the consuming workspace, because the
protocol's reference implementation deliberately keeps no protobuf dependency:
its class bodies are plain Rust structs, and the driver owns the
serialization.

## Regenerating

```bash
npm install
npm run generate          # write the bindings
npm run generate:check    # fail when they are stale
```

The toolchain is pinned through the package's devDependencies, so a checkout
regenerates byte-identical output.

## Adding a field

1. Add it to `proto/mlp/v1/mlp.proto` with the next free tag. **Never reuse a
   tag**, and never renumber an existing one: a field number is a wire
   contract.
2. Regenerate the bindings and commit them.
3. Update the specification's normative tables in the same change.
4. Bump the protocol version if the change is not backward compatible.

## License

Apache-2.0. See [LICENSE](LICENSE).

The reference implementations under
[`mlp-core`](../mlp-core) and [`mlp-ts`](../mlp-ts) are under
PolyForm Noncommercial 1.0.0; this schema is not, so any implementation may
interoperate with them.
