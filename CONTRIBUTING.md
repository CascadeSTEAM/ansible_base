# Contributing to Ansible SOHO Infrastructure

Thank you for your interest in contributing to this project! This guide will help you get started.

## How to Contribute

### Reporting Issues

If you find a bug or have a feature request:
1. Check existing issues first
2. Create a new issue with:
   - Clear description
   - Steps to reproduce (for bugs)
   - Expected vs actual behavior
   - Your environment details (OS, Ansible version, etc.)

### Submitting Changes

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Make your changes
4. Test thoroughly
5. Commit with clear messages
6. Push to your fork
7. Submit a pull request

## Development Guidelines

### Code Style

- **YAML**: Use 2-space indentation
- **Jinja2**: Follow Ansible best practices
- **Comments**: Document complex logic
- **Variables**: Use descriptive names with underscores

### Ansible Best Practices

1. **Idempotency**: Tasks should be safe to run multiple times
2. **Error Handling**: Use `ignore_errors` and `failed_when` appropriately
3. **Conditionals**: Use `when` clauses for conditional execution
4. **Variables**: Use group_vars and host_vars for configuration
5. **Roles**: Keep roles focused and reusable

### Testing

Before submitting:
1. Test syntax: `ansible-playbook --syntax-check playbooks/yourplaybook.yml`
2. Run in check mode: `ansible-playbook --check playbooks/yourplaybook.yml`
3. Test on a non-production system
4. Verify idempotency (run twice, second run should show no changes)

### Directory Structure

```
ansible_base/
├── inventory/
│   ├── dynamic/          # Dynamic inventory plugins
│   ├── group_vars/       # Group variables
│   └── host_vars/        # Host-specific variables
├── playbooks/
│   ├── templates/        # Jinja2 templates
│   └── *.yml            # Playbook files
└── roles/
    └── role_name/
        ├── tasks/        # Role tasks
        ├── defaults/     # Default variables
        ├── handlers/     # Handlers
        ├── templates/    # Role templates
        └── files/        # Static files
```

### Adding New Playbooks

1. Create playbook in `playbooks/` directory
2. Use descriptive name: `verb_noun.yml` (e.g., `install_applications.yml`)
3. Include header comment explaining purpose
4. Document required variables
5. Add error handling
6. Update README.md with usage instructions

### Adding New Roles

1. Create role structure: `ansible-galaxy role init roles/role_name`
2. Implement in `tasks/main.yml`
3. Set defaults in `defaults/main.yml`
4. Add handlers in `handlers/main.yml`
5. Document in role README
6. Test thoroughly

### Adding New Inventory Sources

1. Create configuration in `inventory/dynamic/`
2. Follow plugin documentation
3. Add example configuration
4. Document required credentials/permissions
5. Test discovery

## Pull Request Process

1. **Description**: Clearly describe what your PR does
2. **Testing**: Document how you tested the changes
3. **Breaking Changes**: Note any breaking changes
4. **Documentation**: Update README.md if needed
5. **Review**: Address reviewer feedback

## Commit Messages

Use clear, descriptive commit messages:

```
Add support for Debian systems

- Add Debian-specific package names
- Update conditional checks for Debian family
- Add Debian testing documentation
```

## Questions?

Open an issue for discussion before major changes.

## License

By contributing, you agree that your contributions will be licensed under the same license as the project.
