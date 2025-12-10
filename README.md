# Personal Blog

[![Deploy Hugo site to Pages](https://github.com/pyaethu-aung/personal-blog/actions/workflows/hugo.yaml/badge.svg)](https://github.com/pyaethu-aung/personal-blog/actions/workflows/hugo.yaml)
[![Security](https://github.com/pyaethu-aung/personal-blog/actions/workflows/security.yml/badge.svg)](https://github.com/pyaethu-aung/personal-blog/actions/workflows/security.yml)
[![License](https://img.shields.io/github/license/pyaethu-aung/personal-blog)](https://opensource.org/licenses/MIT)
[![Hugo](https://img.shields.io/badge/Hugo-0.152.2-blue)](https://gohugo.io/)
[![Go](https://img.shields.io/badge/Go-1.25.5-blue)](https://golang.org/)

This project is a personal blog built using [Hugo](https://gohugo.io/) and the [Blowfish](https://blowfish.page/) theme.

## Prerequisites

- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Go](https://go.dev/doc/install)
- [Dart Sass](https://gohugo.io/hugo-pipes/transpile-sass-to-css/#dart-sass) (optional)

## Installation

1. Clone the repository:
    ```bash
    git clone https://github.com/yourusername/personal-blog.git
    cd personal-blog
    ```

2. Install Hugo:
    ```bash
    brew install hugo
    ```

3. Tidy Hugo modules:
    ```bash
    hugo mod tidy
    ```

4. Download configuration files for Blowfish:
    ```bash
    curl -L https://minhaskamal.github.io/DownGit/#/home?url=https://github.com/nunocoracao/blowfish/tree/main/config/_default -o config.zip
    unzip config.zip -d config/_default
    ```

5. Add Blowfish as a Hugo module:
    ```toml
    # config/_default/module.toml
    [[imports]]
    disable = false
    path = "github.com/nunocoracao/blowfish/v2"
    ```

## Running the Project Locally

1. Start the Hugo server:
    ```bash
    hugo server --port 8080
    ```

2. Open your browser and navigate to [http://localhost:8080/](http://localhost:8080/).

## Deployment

This project uses GitHub Actions to deploy the site to GitHub Pages. The workflow is defined in [`.github/workflows/hugo.yaml`](.github/workflows/hugo.yaml).

## Contributing

Feel free to open issues or submit pull requests for any improvements or bug fixes.

### Branch Naming Convention

This project follows a structured branch naming convention to maintain clarity and organization. All branches must follow this format:

```
prefix/description
```

**Valid prefixes:**
- `feature/` - for new features
- `bugfix/` - for bug fixes
- `hotfix/` - for hotfixes
- `refactor/` - for refactoring
- `docs/` - for documentation
- `test/` - for tests
- `ci/` - for CI/CD updates
- `chore/` - for maintenance and dependencies

**Examples:**
- `feature/add-dark-mode`
- `bugfix/fix-navigation`
- `docs/update-readme`
- `docs/installation-guide`
- `ci/update-hugo-workflow`
- `chore/update-dependencies`
- `chore/blowfish-theme`

**Note:** Branch names must use lowercase letters, numbers, hyphens, and underscores only.

## License

This project is licensed under the [MIT License](LICENSE).
