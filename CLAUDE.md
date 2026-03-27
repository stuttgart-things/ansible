# Ansible Collections Repository

## Project Structure

- `collections/` — Source definitions for Ansible collections (baseos, container, awx, rke)
- Each collection has a `collection.yaml` (metadata) and one or more YAML files defining playbooks

## Collection Build System

Collections are **not standard Ansible layout on disk**. They use a custom DSL in YAML files that gets processed by a **Dagger Go module** (`github.com/stuttgart-things/dagger/ansible`) into a proper Ansible collection structure.

### How it works

Each YAML file (e.g. `setup.yaml`) can contain:
- `play:` — inline playbook YAML
- `templates:` — list of `{name, file}` entries for Jinja2 templates
- `vars:` — list of `{name, file}` entries for variable files

The Dagger module processes these into:
```
playbooks/templates/{name}.yaml   # template files
playbooks/vars/{name}.yaml        # var files
playbooks/{playbook}.yaml         # playbook files
```

### Critical: Template file naming

The build system **appends `.yaml`** to template names. So a template named `starship.toml.j2` becomes `starship.toml.j2.yaml` on disk. Playbook `src:` references must include the `.yaml` suffix:
```yaml
# WRONG
src: starship.toml.j2

# CORRECT
src: starship.toml.j2.yaml
```

### Build commands

```bash
task build-collection   # Interactive collection build (uses gum to select)
task release            # Commit, create PR, merge
```

Build pipeline stages (all via Dagger):
1. `init-collection` — Parse source YAML, create collection structure
2. `modify-role-includes` — Adjust role includes
3. `build` — Produce `.tar.gz` artifact and install via `ansible-galaxy`

## Conventions

- Namespace: `sthings`
- Collections: baseos, container, awx, rke
- Task runner: [Taskfile](https://taskfile.dev/) (`Taskfile.yaml`)
- Testing: Molecule (requires venv: `task setup-venv && source .venv/bin/activate && task setup-molecule`)
