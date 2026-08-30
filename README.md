# onyx-server

Server for the Onyx Knowledge Management System.

This package defines the Onyx **wire protocol** and ships a basic **reference server**
that implements it.

## What's in here

- **Wire protocol** — the message format and request/response semantics spoken between
  Onyx clients and servers. This is the primary deliverable of the package.
- **Reference server** — a minimal implementation that serves a vault from the local
  filesystem.

## Vault model

A vault is a directory of Obsidian-style Markdown files:

- Notes are plain `.md` files, addressed by their path relative to the vault root.
- Obsidian conventions apply — `[[wikilinks]]`, YAML frontmatter, attachments alongside notes.
- **Git** is the version control layer. History, diffs, and sync are delegated to the
  repository containing the vault rather than reimplemented by the server.

## Running the server

The server takes a path to the vault root and serves every file under it. The path
defaults to the current working directory:

```sh
# serve $PWD
zig build run

# serve an explicit vault path
zig build run -- /path/to/vault
```

## Consuming the protocol

The wire protocol is exposed as a Zig build system module, so clients can depend on this
package and share the protocol definitions instead of duplicating them.

In your `build.zig.zon`:

```sh
zig fetch --save git+https://github.com/StaffinityAI/onyx-server
```

In your `build.zig`:

```zig
const onyx = b.dependency("onyx_server", .{
    .target = target,
    .optimize = optimize,
});

exe.root_module.addImport("onyx-protocol", onyx.module("onyx-protocol"));
```

## Clients

- [onyx-plugin](https://github.com/StaffinityAI/onyx-plugin) — first client.

## License

MIT. See [LICENSE](LICENSE).
