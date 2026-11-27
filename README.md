# Domus

The household apps of one author and what they share, in one
repository:

- [Mixtura](apps/mixtura) (Drink Builder): a workbench for mixed drinks.
- [Culina](apps/culina) (Cook's Notebook): a daily driver to get ideas, click dishes together, see what goes well together and the taste profile of the dish.
- [Emptio](apps/emptio) (Shopping List): the household's shopping list.
- [Household](apps/domus): the app at the root of the address where sync is set up once for all apps (spaces, storages, invites).
- [materia](packages/model): the general model the apps share.
- [hospes](packages/host): the host code the apps share (sync cipher, WebDAV adapter).

Free software, offline first, no account and no server of ours. Why one
repository: [docs/decisions/one-repository.md](docs/decisions/one-repository.md).

## Development

```sh
npm ci
```

```sh
npm run check
```

`npm run check:mixtura` (or `:culina`, `:emptio`, `:domus`, `:model`, `:host`) checks one
part. More in [docs/development.md](docs/development.md).

## Licence

AGPL-3.0-or-later, see [LICENSE](LICENSE).
