# DevJournal

A small multi-author blog (a "dev journal") built to learn Laravel. The web UI is served via Inertia + React; a separate REST API exposes the same domain as JSON for external clients.

## Language

**User**:
A person who owns an account. The single identity entity — registers, logs in, and may act as an author or commenter. This is the only account model.
_Avoid_: Account, member, writer

**Author**:
The role a User plays when they own a Post. Not a separate model — it is the name of the relationship from a Post back to its owning User (`$post->author`).
_Avoid_: Writer, creator

**Post**:
A blog article written by a User. Owned by exactly one User (its author).
_Avoid_: Article, entry, blog

**Comment**:
A reply written by a logged-in User on a Post. Always belongs to both a User and a Post — guests cannot comment.
_Avoid_: Reply, message

**Tag**:
A free-form label attached to Posts to group them by topic. A Post can have many Tags and a Tag applies to many Posts.
_Avoid_: Category, label, topic

**Published**:
A Post whose `published_at` is set and not in the future. Visible to everyone, including guests and the REST API.
_Avoid_: Live, public, active

**Draft**:
A Post whose `published_at` is null. Visible only to its author, never to guests or the REST API.
_Avoid_: Unpublished, hidden, private
