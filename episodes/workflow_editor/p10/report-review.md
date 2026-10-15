# p10 review correction — oversized IO records and report accuracy

## Changes

- Fixed IO delivery stopping at a JSON log line larger than the normal 2 MiB
  read batch. The reader now assembles that one line through its newline,
  within the file size observed when the pass started. It then uses the same
  public-output extraction, credential redaction, excerpt and full-entry view
  as ordinary records. The next pass continues after that line.
- Incomplete lines remain unconsumed until their newline arrives. Chunks are
  decoded together so UTF-8 characters crossing a read boundary stay intact.
  Memory scales with one oversized line; the batch budget is not a memory cap.
- Corrected the final report's obsolete claim that old agforge retirement
  awaited the Developer. A fresh nctl drift observation shows its service
  inactive with no active placement. The unrelated Observer service aggregate
  still reports `service_missing`.
- Moved host-specific inspection addresses out of the final and deployment
  reports into ignored `pj-agdev/.local/devenv.md`, under *workflow_editor p10
  review correction*. The retained run identities and navigation instructions
  still identify the parent, child, request and detailed IO.

## Verification

Both new regression tests failed against the original reader and pass with
the fix:

1. A line larger than the read batch, containing multibyte characters across
   chunk boundaries, is not consumed prematurely and preserves its exact text
   once its newline arrives. The following line remains readable.
2. Through the notifier, a tool result larger than 2 MiB is delivered with its
   full-entry link. The full entry matches the original output, later public
   output and the attempt's end boundary are delivered, the stored cursor
   reaches the last line, and reopening the notifier does not replay anything.

The IO suite passes 8/8 and the standalone type check passes.

Source: agdevworld `399e2b2`; superproject pointer: pj-agdev `b9231c6`.

## Deployment

The committed release passed `npm run check`: **337/337 tests**, type check
and production build. It was then activated using the normal release script;
the service and CLI both report `399e2b2`. No store migration or executor code
change was needed.

After activation, the LAN editor returns HTTP 200, the existing forge trial's
IO channel and all three ended-attempt cursors remain accessible, the executor
is healthy and idle with no queued jobs, and observations report no source
problems. The previously held experiment remains stopped at sequence 11.
Exact addresses and the build log location remain in ignored local notes.
