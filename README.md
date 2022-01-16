# composer-plugin-issue

This issue actually harks back to [an earlier one I reported][1] almost two
years ago. The root cause is that plugins and scripts required by a project
will use libraries that are bundled in `composer.phar`, rather than from the
project's vendor directory.

Let's examine this issue with a Composer plugin.

I've created [ramsey/composer-repl][2], which requires [symfony/console][3].
I recently began working on a project that requires version `^6.0` of
symfony/console. I attempted to require ramsey/composer-repl, but due to its
dependency constraints, Composer would not allow installation. So, I updated
ramsey/composer-repl to allow installation with projects that use version 6 of
Symfony packages.

So, now, I can install ramsey/composer-repl in a project that requires
symfony/console version 6. Nothing in ramsey/composer-repl conflicts with any
changes to the API for symfony/console, and even if it did, I can control these
changes with conditionals.

However, ramsey/composer-repl is a Composer plugin, so it doesn't use the
symfony/console code that's installed in `vendor/`. Rather, it uses the
symfony/console code that's bundled in `composer.phar`.

As a result, when I run `composer repl`, I get the following error:

```
PHP Fatal error:  Declaration of Symfony\Component\Console\Input\ArrayInput::hasParameterOption(array|string $values, bool $onlyParams = false): bool must be compatible with Symfony\Component\Console\Input\InputInterface::hasParameterOption($values) in /path/to/composer-plugin-issue/vendor/symfony/console/Input/ArrayInput.php on line 56
```

## Steps to Reproduce

Make sure you have the latest version of Composer (v2.2.4) installed.

I'm running this on PHP 8.1.1.

```shell
git clone https://github.com/ramsey/composer-plugin-issue.git
cd composer-plugin-issue/
composer install
composer repl
```

You should see the following error occur after running `composer repl`:

```
PHP Fatal error:  Declaration of Symfony\Component\Console\Input\ArrayInput::hasParameterOption(array|string $values, bool $onlyParams = false): bool must be compatible with Symfony\Component\Console\Input\InputInterface::hasParameterOption($values) in /path/to/composer-plugin-issue/vendor/symfony/console/Input/ArrayInput.php on line 56

Fatal error: Declaration of Symfony\Component\Console\Input\ArrayInput::hasParameterOption(array|string $values, bool $onlyParams = false): bool must be compatible with Symfony\Component\Console\Input\InputInterface::hasParameterOption($values) in /path/to/composer-plugin-issue/vendor/symfony/console/Input/ArrayInput.php on line 56
```

Note that the error is not related to any code in ramsey/composer-repl, but it
occurs because ramsey/composer-repl uses symfony/console, and the local project
has symfony/console v6 installed, while [Composer has version 2.8.52][4]
bundled with it.

It appears, PHP is attempting to use the symfony/console v6 version of
`InputInterface` with the v2.8.52 version of `ArrayInput`, and this results in
a fatal error.

### Using Composer Snapshots

I also tried the above using the latest Composer snapshot:

```
❯ composer self-update --snapshot
Upgrading to version 96332161feb920e56280ac99af5d548dc3e25b51 (snapshot channel).
```

This results in the same fatal error.

[1]: https://github.com/composer/composer/issues/8907
[2]: https://github.com/ramsey/composer-repl
[3]: https://github.com/symfony/console
[4]: https://github.com/composer/composer/blob/2.2.4/composer.lock#L733
