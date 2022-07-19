# pre-commit-composer

A [pre-commit](https://pre-commit.com/) hook for [Composer](https://github.com/composer/composer).

This hook runs [composer validate](https://getcomposer.org/doc/03-cli.md#validate) when committing code that changes `composer.json` or `composer.lock`.

## Usage

Add to your `.pre-commit-config.yaml`:

```yml
-   repo: https://github.com/fengtan/pre-commit-composer
    rev: 1.0.0 # Use the ref you want
    hooks:
    -   id: composer-validate
        args: ["--strict"] # Optional ; set any option you want from https://getcomposer.org/doc/03-cli.md#validate
```

This hook expects `composer` to be locally installed.

## How to test it

**1. Get an environment with Composer and Pre-commit**

```sh
$ docker run --rm -it php:7.4-cli /bin/sh
$ apt-get update && apt-get install --yes git wget unzip pip vim
$ wget https://raw.githubusercontent.com/composer/getcomposer.org/76a7060ccb93902cd7576b67264ad91c8a2700e2/web/installer -O - -q | php -- --quiet && mv composer.phar /usr/local/bin/composer
$ pip install pre-commit
$ git config --global user.email "foo@example.com"
$ git config --global user.name "Foo Bar"
```

**2. Create a Composer project, initialize a Git repo, add our pre-commit hook**

```sh
$ composer create-project laravel/laravel laravel 4.2.* --no-plugins
$ cd laravel/
$ git init
$ cat <<EOT >> .pre-commit-config.yaml
repos:
-   repo: https://github.com/fengtan/pre-commit-composer
    rev: 1.0.0
    hooks:
    -   id: composer-validate
EOT
$ pre-commit install
```

**3. Simulate an invalid `composer.lock`**

```
$ echo garbage >> composer.lock
```

**4. Commit changeset: the hook will run and fail because of the invalid `composer.lock` ; the commit will abort**

```
$ git add -A
$ git commit
composer validate........................................................Failed
- hook id: composer-validate
- exit code: 1

In JsonFile.php line 349:
                                                 
  "./composer.lock" does not contain valid JSON  
  Parse error on line 2151:                      
  ...version": "2.3.0"}garbage                   
  --------------------^                          
  Expected one of: 'EOF', '}', ',', ']'          
```
