# onyx-server

Server for the Onyx Knowledge Management System.

This package defines the Onyx **wire protocol** and ships a basic **reference server**
that implements it.

## What's in here

- **Wire protocol** — the message format and request/response semantics spoken between
  Onyx clients and servers. This is the primary deliverable of the package.
- **Reference server** — a minimal implementation that serves a vault from the local
  filesystem and owns the collaborative editing state for it.

## Vault model

A vault is a directory of Obsidian-style Markdown files:

- Notes are plain `.md` files, addressed by their path relative to the vault root.
- Obsidian conventions apply — `[[wikilinks]]`, YAML frontmatter, attachments alongside notes.
- **Git** is the version control layer. History, diffs, and sync are delegated to the
  repository containing the vault rather than reimplemented by the server.

## Collaboration

Collaboration is a core concern of the protocol, not a layer bolted on top of it.

**The server owns one central CRDT per document.** All editing state lives there. Clients
are deliberately **dumb views**: they render the document, send local intents (edits,
cursor and selection positions) to the server, and apply whatever the server sends back.
They do not run their own CRDT, do not merge concurrent edits, and hold no authoritative
copy of the document.

Consequences of that split:

- Convergence is trivial — there is exactly one replica that matters, so there is no
  client-to-client reconciliation and no split-brain to repair.
- Presence rides the same channel as edits. The server broadcasts every participant's
  cursor and selection, so clients can always show all cursors in a document.
- A client can be thin. Implementing a new client means speaking the wire protocol and
  drawing text — not shipping a CRDT.

## Persistence and saving

The in-memory CRDT is the live document; **git is the commit history**.

Clients do not write files. A client sends a **save request** over the wire protocol, and
the server serializes the current CRDT state to Markdown on disk and runs a **git commit**
in the vault repository. Saving is therefore an explicit, shared, versioned checkpoint of
the collaborative state rather than a per-client flush.

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
