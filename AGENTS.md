# Creating posts from the inbox

The repository's inbox is `input/` (sometimes referred to as `/inbox`).

After creating a post from files in this folder:

1. Verify that the post body contains the supplied text and that each used image
   has been copied into the post's page bundle and is referenced correctly.
2. Delete the source files used for that post from `input/` only after those
   checks succeed. Keep the folder and any files not used for that post.
3. If creation or verification fails, preserve the source files.

This cleanup is part of creating the post; it does not need to wait for a Git
commit or deployment.
