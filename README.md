<p align="center">
  <a href="https://sentry.io/?utm_source=github&utm_medium=logo" target="_blank">
    <img src="https://sentry-brand.storage.googleapis.com/sentry-wordmark-dark-280x84.png" alt="Sentry" width="280" height="84">
  </a>
</p>

# Sentry Vite Plugin

[![npm version](https://img.shields.io/npm/v/@sentry/vite-plugin.svg)](https://www.npmjs.com/package/@sentry/vite-plugin)
[![npm dm](https://img.shields.io/npm/dm/@sentry/vite-plugin.svg)](https://www.npmjs.com/package/@sentry/vite-plugin)
[![npm dt](https://img.shields.io/npm/dt/@sentry/vite-plugin.svg)](https://www.npmjs.com/package/@sentry/vite-plugin)

> A Vite plugin that provides source map and release management support for Sentry.

## Installation

Using npm:

```bash
npm install @sentry/vite-plugin --save-dev
```

Using yarn:

```bash
yarn add @sentry/vite-plugin --dev
```

Using pnpm:

```bash
pnpm add -D @sentry/vite-plugin
```

## Example

```ts
// vite.config.ts
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import { sentryVitePlugin } from "@sentry/vite-plugin";

// https://vitejs.dev/config/
export default defineConfig({
  build: {
    sourcemap: true, // Source map generation must be turned on
  },
  plugins: [
    vue(),

    // Put the Sentry vite plugin after all other plugins
    sentryVitePlugin({
      org: process.env.SENTRY_ORG,
      project: process.env.SENTRY_PROJECT,

      // Auth tokens can be obtained from https://sentry.io/orgredirect/organizations/:orgslug/settings/auth-tokens/
      authToken: process.env.SENTRY_AUTH_TOKEN,
    }),
  ],
});
```

## Options

-   [`org`](#org)
-   [`project`](#project)
-   [`authToken`](#authtoken)
-   [`url`](#url)
-   [`headers`](#headers)
-   [`debug`](#debug)
-   [`silent`](#silent)
-   [`errorHandler`](#errorhandler)
-   [`telemetry`](#telemetry)
-   [`disable`](#disable)
-   [`sourcemaps`](#sourcemaps)
    -   [`assets`](#sourcemapsassets)
    -   [`ignore`](#sourcemapsignore)
    -   [`rewriteSources`](#sourcemapsrewritesources)
    -   [`filesToDeleteAfterUpload`](#sourcemapsfilestodeleteafterupload)
-   [`release`](#release)
    -   [`name`](#releasename)
    -   [`inject`](#releaseinject)
    -   [`create`](#releasecreate)
    -   [`finalize`](#releasefinalize)
    -   [`dist`](#releasedist)
    -   [`vcsRemote`](#releasevcsremote)
    -   [`setCommits`](#releasesetcommits)
    -   [`deploy`](#releasedeploy)
    -   [`cleanArtifacts`](#releasecleanartifacts)
    -   [`uploadLegacySourcemaps`](#releaseuploadlegacysourcemaps)
-   [`_experiments`](#_experiments)
    -   [`injectBuildInformation`](#_experimentsinjectbuildinformation)

### `org`

Type: `string`

The slug of the Sentry organization associated with the app.

This value can also be specified via the `SENTRY_ORG` environment variable.

### `project`



The slug of the Sentry project associated with the app.

This value can also be specified via the `SENTRY_PROJECT` environment variable.

### `authToken`

Type: `string`

The authentication token to use for all communication with Sentry. Can be obtained from https://sentry.io/orgredirect/organizations/:orgslug/settings/auth-tokens/.

This value can also be specified via the `SENTRY_AUTH_TOKEN` environment variable.

### `url`

Type: `string`

The base URL of your Sentry instance. Use this if you are using a self-hosted or Sentry instance other than sentry.io.

This value can also be set via the SENTRY_URL environment variable.

Defaults to https://sentry.io/, which is the correct value for SaaS customers.

### `headers`

Type: `Record<string, string>`

Headers added to every outgoing network request.

### `debug`

Type: `boolean`

Print useful debug information. Defaults to `false`.

### `silent`

Type: `boolean`

Suppresses all logs. Defaults to `false`.

### `errorHandler`

Type: `(err: Error) => void`

When an error occurs during release creation or sourcemaps upload, the plugin will call this function.

By default, the plugin will simply throw an error, thereby stopping the bundling process. If an `errorHandler` callback is provided, compilation will continue, unless an error is thrown in the provided callback.

To allow compilation to continue but still emit a warning, set this option to the following:

```
errorHandler: (err) => {
  console.warn(err);
}
```


### `telemetry`

Type: `boolean`

If set to true, internal plugin errors and performance data will be sent to Sentry.

At Sentry we like to use Sentry ourselves to deliver faster and more stable products. We're very careful of what we're sending. We won't collect anything other than error and high-level performance data. We will never collect your code or any details of the projects in which you're using this plugin.

Defaults to `true`.

### `disable`

Type: `boolean`

Completely disables all functionality of the plugin. Defaults to `false`.

### `sourcemaps`



Options for uploading source maps.
### `sourcemaps.assets`

Type: `string | string[]`

A glob or an array of globs that specifies the build artifacts that should be uploaded to Sentry.

If this option is not specified, the plugin will try to upload all JavaScript files and source map files that are created during build.

The globbing patterns follow the implementation of the `glob` package. (https://www.npmjs.com/package/glob)

Use the `debug` option to print information about which files end up being uploaded.

### `sourcemaps.ignore`

Type: `string | string[]`

A glob or an array of globs that specifies which build artifacts should not be uploaded to Sentry.

Default: `[]`

The globbing patterns follow the implementation of the `glob` package. (https://www.npmjs.com/package/glob)

Use the `debug` option to print information about which files end up being uploaded.

### `sourcemaps.rewriteSources`

Type: `(source: string, map: any) => string`

Hook to rewrite the `sources` field inside the source map before being uploaded to Sentry. Does not modify the actual source map. Effectively, this modifies how files inside the stacktrace will show up in Sentry.

Defaults to making all sources relative to `process.cwd()` while building.

### `sourcemaps.filesToDeleteAfterUpload`

Type: `string | string[]`

A glob or an array of globs that specifies the build artifacts that should be deleted after the artifact upload to Sentry has been completed.

The globbing patterns follow the implementation of the `glob` package. (https://www.npmjs.com/package/glob)

Use the `debug` option to print information about which files end up being deleted.

### `release`



Options related to managing the Sentry releases for a build.

More info: https://docs.sentry.io/product/releases/
### `release.name`

Type: `string`

Unique identifier for the release you want to create.

This value can also be specified via the `SENTRY_RELEASE` environment variable.

Defaults to automatically detecting a value for your environment. This includes values for Cordova, Heroku, AWS CodeBuild, CircleCI, Xcode, and Gradle, and otherwise uses the git `HEAD`'s commit SHA. (the latterrequires access to git CLI and for the root directory to be a valid repository)

If you didn't provide a value and the plugin can't automatically detect one, no release will be created.

### `release.inject`

Type: `boolean`

Whether the plugin should inject release information into the build for the SDK to pick it up when sending events. (recommended)

Defaults to `true`.

### `release.create`

Type: `boolean`

Whether the plugin should create a release on Sentry during the build. Note that a release may still appear in Sentry even if this is value is `false` because any Sentry event that has a release value attached will automatically create a release. (for example via the `inject` option)

Defaults to `true`.

### `release.finalize`

Type: `boolean`

Whether the Sentry release should be automatically finalized (meaning an end timestamp is added) after the build ends.

Defaults to `true`.

### `release.dist`

Type: `string`

Unique identifier for the distribution, used to further segment your release.

### `release.vcsRemote`

Type: `string`

Version control system remote name.

This value can also be specified via the `SENTRY_VSC_REMOTE` environment variable.

Defaults to 'origin'.

### `release.setCommits`



Option to associate the created release with its commits in Sentry.
### `release.setCommits.previousCommit`

Type: `string`

The commit before the beginning of this release (in other words, the last commit of the previous release).

Defaults to the last commit of the previous release in Sentry.

If there was no previous release, the last 10 commits will be used.

### `release.setCommits.ignoreMissing`

Type: `boolean`

If the flag is to `true` and the previous release commit was not found in the repository, the plugin creates a release with the default commits count instead of failing the command.

Defaults to `false`.

### `release.setCommits.ignoreEmpty`

Type: `boolean`

If this flag is set, the setCommits step will not fail and just exit silently if no new commits for a given release have been found.

Defaults to `false`.

### `release.setCommits.auto`

Type: `boolean`

Automatically sets `commit` and `previousCommit`. Sets `commit` to `HEAD` and `previousCommit` as described in the option's documentation.

If you set this to `true`, manually specified `commit` and `previousCommit` options will be overridden. It is best to not specify them at all if you set this option to `true`.

### `release.setCommits.repo`

Type: `string`

The full repo name as defined in Sentry.

Required if the `auto` option is not set to `true`.

### `release.setCommits.commit`

Type: `string`

The current (last) commit in the release.

Required if the `auto` option is not set to `true`.

### `release.deploy`



Adds deployment information to the release in Sentry.
### `release.deploy.env`

Type: `string`

Environment for this release. Values that make sense here would be `production` or `staging`.

### `release.deploy.started`

Type: `number | string`

Deployment start time in Unix timestamp (in seconds) or ISO 8601 format.

### `release.deploy.finished`

Type: `number | string`

Deployment finish time in Unix timestamp (in seconds) or ISO 8601 format.

### `release.deploy.time`

Type: `number`

Deployment duration (in seconds). Can be used instead of started and finished.

### `release.deploy.name`

Type: `string`

Human readable name for the deployment.

### `release.deploy.url`

Type: `string`

URL that points to the deployment.

### `release.cleanArtifacts`

Type: `boolean`

Remove all previously uploaded artifacts for this release on Sentry before the upload.

Defaults to `false`.

### `release.uploadLegacySourcemaps`

Type: `string | IncludeEntry | Array<string | IncludeEntry>`

Legacy method of uploading source maps. (not recommended unless necessary)
One or more paths that should be scanned recursively for sources.

Each path can be given as a string or an object with more specific options.

The modern version of doing source maps upload is more robust and way easier to get working but has to inject a very small snippet of JavaScript into your output bundles.
In situations where this leads to problems (e.g subresource integrity) you can use this option as a fallback.

The `IncludeEntry` type looks as follows:

```ts
type IncludeEntry = {
    /**
     * One or more paths to scan for files to upload.
     */
    paths: string[];

    /**
     * One or more paths to ignore during upload.
     * Overrides entries in ignoreFile file.
     *
     * Defaults to `['node_modules']` if neither `ignoreFile` nor `ignore` is set.
     */
    ignore?: string | string[];

    /**
     * Path to a file containing list of files/directories to ignore.
     *
     * Can point to `.gitignore` or anything with the same format.
     */
    ignoreFile?: string;

    /**
     * Array of file extensions of files to be collected for the file upload.
     *
     * By default the following file extensions are processed: js, map, jsbundle and bundle.
     */
    ext?: string[];

    /**
     * URL prefix to add to the beginning of all filenames.
     * Defaults to '~/' but you might want to set this to the full URL.
     *
     * This is also useful if your files are stored in a sub folder. eg: url-prefix '~/static/js'.
     */
    urlPrefix?: string;

    /**
     * URL suffix to add to the end of all filenames.
     * Useful for appending query parameters.
     */
    urlSuffix?: string;

    /**
     * When paired with the `rewrite` option, this will remove a prefix from filename references inside of
     * sourcemaps. For instance you can use this to remove a path that is build machine specific.
     * Note that this will NOT change the names of uploaded files.
     */
    stripPrefix?: string[];

    /**
     * When paired with the `rewrite` option, this will add `~` to the `stripPrefix` array.
     *
     * Defaults to `false`.
     */
    stripCommonPrefix?: boolean;

    /**
     * Determines whether sentry-cli should attempt to link minified files with their corresponding maps.
     * By default, it will match files and maps based on name, and add a Sourcemap header to each minified file
     * for which it finds a map. Can be disabled if all minified files contain sourceMappingURL.
     *
     * Defaults to true.
     */
    sourceMapReference?: boolean;

    /**
     * Enables rewriting of matching source maps so that indexed maps are flattened and missing sources
     * are inlined if possible.
     *
     * Defaults to true
     */
    rewrite?: boolean;

    /**
     * When `true`, attempts source map validation before upload if rewriting is not enabled.
     * It will spot a variety of issues with source maps and cancel the upload if any are found.
     *
     * Defaults to `false` as this can cause false positives.
     */
    validate?: boolean;
};
```



### `_experiments`

Type: `string`

Options that are considered experimental and subject to change. This option does not follow semantic versioning and may change in any release.
### `_experiments.injectBuildInformation`

Type: `boolean`

If set to true, the plugin will inject an additional `SENTRY_BUILD_INFO` variable. This contains information about the build, e.g. dependencies, node version and other useful data.

Defaults to `false`.



### Configuration File

As an additional configuration method, the Sentry Vite plugin will pick up environment variables configured inside a `.env.sentry-build-plugin` file located in the current working directory when building your app.

## More information

- [Sentry Documentation](https://docs.sentry.io/quickstart/)
- [Sentry Discord](https://discord.gg/Ww9hbqr)
- [Sentry Stackoverflow](http://stackoverflow.com/questions/tagged/sentry)
