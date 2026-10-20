# Catalog signing keys

The public halves of the Ed25519 keys that may sign this catalog's
[store API](../docs/api.md#verifying-the-catalog). A catalog made from the template starts with
none and is published unsigned; ProsperoStore then reads it with **Verify custom catalog
signatures** switched off. Signing is optional until stores can trust a catalog's own keys.

To sign your catalog:

1. Make a key pair:

   ```sh
   openssl genpkey -algorithm ed25519 -out catalog-signing-1.pem
   openssl pkey -in catalog-signing-1.pem -pubout -out keys/catalog-signing-1.pub.pem
   ```

2. Commit `keys/catalog-signing-1.pub.pem`. Never commit the private key.
3. In the repository's **Settings → Environments**, open the `cloudflare-pages` environment
   (the deploy job creates it on its first run) and add the secret `CATALOG_SIGNING_KEY` with
   the contents of `catalog-signing-1.pem`.

The deploy job signs the manifest with that key and refuses a key whose public half isn't
in this folder. Keys have no expiry; changing this folder changes which catalogs clients
accept, so treat it like a release.
