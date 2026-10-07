# Archer

Archer is a multilingual freelance marketplace for clients and freelancers. This repository coordinates three independently versioned projects through Git submodules.

## Projects

- [`archer-app`](https://github.com/paingkhant-fsd/archer-app) - React web client
- [`archer-api`](https://github.com/paingkhant-fsd/archer-api) - Express API and Prisma database layer
- [`archer-mobile`](https://github.com/paingkhant-fsd/archer-mobile) - Expo and React Native client

Product behavior is defined in [`SPEC.md`](./SPEC.md). Cross-project engineering boundaries are defined in [`AGENTS.md`](./AGENTS.md). Each project README owns its installation, environment, and run instructions.

## Prerequisites

- Git with submodule support
- Node.js 24+
- npm 11+

Platform-specific prerequisites are documented by each project.

## Clone

Clone the complete workspace and initialize every project:

```powershell
git clone --recurse-submodules https://github.com/paingkhant-fsd/archer.git
cd archer
```

If the repository was cloned without submodules, initialize them afterward:

```powershell
git submodule update --init --recursive
```

Then follow the setup instructions in:

- [`archer-api/README.md`](./archer-api/README.md)
- [`archer-app/README.md`](./archer-app/README.md)
- [`archer-mobile/README.md`](./archer-mobile/README.md)

Start the API before running a client that needs live marketplace or authentication data.

## Repository boundaries

The three projects are separate repositories. They communicate only through the versioned `/api/v1` HTTP contract and do not import each other's source code or internal packages.

When updating a submodule revision in this repository, commit and push the child project first, then commit the resulting gitlink change here.

## License

Archer is available under the [MIT License](./LICENSE).
