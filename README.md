# OBS Source Service `obs-service-publish_tar`

`obs-service-publish_tar` will create a `archive.tar[.<tar compression>]` archive
containing the `published repository`. The archive is generated in the rpm package directory.

This can be useful if you need to package and distribute a snapshot of a project's packages in a ready-to-deploy archive.

## Usage

Place the `publish_tar` and `publish_tar.service` files in /usr/lib/obs/service.

On your package in OBS, create a \_service file with the following contents:

```
<services>
  <service name="publish_tar">
    <param name="destinationpath">/srv/www/updates</param>
    <param name="compression">gz</param>
    <param name="archive">updates-%d.tar.gz</param>
    <param name="changesgenerate">enable</param>
    <param name="sourceproject">MYPROJECT</param>
    <param name="repository">SLE_15_SP2</param>
    <param name="publish_url">rsync://myserver/reposync/update-archives</param>
    <param name="obs_server">my_obs_server</param>
    <param name="obs_user">my_api_user</param>
    <param name="obs_password">my_api_password</param>
    <param name="keeplatest">enable</param>
    <param name="extrapackages">external.list</param>
  </service>
</services> 
```

Clicking on "trigger services" should do these steps, in order:

* gather the complete published repository for the project indicated in "sourceproject" for the repository indicated in "repository" (except the src and repocache directories)

* compress the contents into a new archive named in "archive", with compression type defined in "compression". A variable of %d will expand to "YYYY-MM-DD". The contents will be under the path defined by "destinationpath" inside the archive (e.g. everything under /srv/www/updates in this example)

* if "changesgenerate" is set to "enable", go around all the packages in the project indicated, look for the corresponding ".changes" file and concatenate them into a global changelog. The changelog will have the same name as the archive, but with a ".changelog" extension. This uses the OBS API server, user and password indicated in the corresponding parameters.

* if a download manifest is specified with "extrapackages", these URLs will be downloaded and included inside a directory also named "extrapackages" at the destination path inside the archive. This should point to simple text file, with one URL on each line. If the downloaded files are RPM files, they'll be included in the repodata. Self-signed certificates are ignored by default.

* if "keeplatest" is set to "enable", an exclusion list will be generated before creating the final tarball, so only the LATEST VERSION of each package is included. The metadata will be regenerated at the end of the process. This is useful when inheriting packages via _aggregate, as it might bring multiple versions of the same package. This also excludes drpm (delta RPMs) and src (source RPMs) from the final archive.

* the  "maxversions" parameter allows one to specify a list with exact versions of packages to include. For example, these lines would pin the versions of these 3 packages to the specified ones. Newer (and older) versions will be ignored:

```
MozillaFirefox,91.12.0-150200.152.53.1
MozillaFirefox-branding-SLE,91-9.5.1
MozillaFirefox-translations-common,91.12.0-150200.152.53.1
```
* the "extrapackages" parameter allows one to include packages to be downloaded and placed on a special directory "extrapackages" inside the tarball. Any URLs that urllib3/curl understands is valid, one per line. These will also be referenced in the repodata.

* finally, it'll publish the archive to the RSYNC server indicated in "publish_url", if present. If an archive already exists with the same name, it'll be overwritten. If you don't specify the "publish_url", it'll just create the tarball in the current package directory. Currently this option uses only RSYNC URLs, like "rsync://myreposerver/reposync/archives", so the corresponding RSYNC module must be configured first at the destination server. This is intended for private OBS installations only.

A publish URL may also be defined in the project config, in this format:

```
%define publish_url rsync://myserver/myrsyncmodule/mydestdir
```

Note that the _service parameter takes precedence, if present.



## License

GNU General Public License v2.0 or later
