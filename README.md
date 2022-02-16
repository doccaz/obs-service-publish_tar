# OBS Source Service `obs-service-publish_tar`

`obs-service-publish_tar` will create a `archive.tar[.<tar compression>]` archive
containing the `published repository`. The archive
is generated in the rpm package directory.

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
  </service>
</services> 
```

Clicking on "trigger services" should do these steps, in order:
* gather the complete published repository for the project indicated in "sourceproject" for the repository indicated in "repository" (except the src and repocache directories)
* compress the contents into a new archive named in "archive", with compression type defined in "compression". A variable of %d will expand to "YYYY-MM-DD".
* the contents will be under the path defined by "destinationpath" inside the archive (e.g. everything under /srv/www/updates in this example)
* if "changesgenerate" is set to "enable", go around all the packages in the project indicated, look for .changes files and concatenate them into a global changelog. The changelog will have the same name as the archive, but with a ".changelog" extension. This uses the OBS API server, user and password indicated in the corresponding parameters.
* if "keeplatest" is set to "enable", an exclusion list will be generated before creating the final tarball, so only the LATEST VERSION of each package is included. The metadata is not regenerated, though. This is useful when inheriting packages via _aggregate, as it might bring dozens of versions of the same package. This also excludes drpm (delta RPMs) and src (source RPMs) from the tarball.
* finally, it'll publish the archive to the RSYNC server indicated in "publish_url". By default we do not erase previous archives.



## License

GNU General Public License v2.0 or later
