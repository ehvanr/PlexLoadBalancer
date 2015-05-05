Plex Load Balancer
=================

NOTE: This is super super beta.  I did have it running without issue for 
roughly a day but just recently I started running into issues with the slave 
complaining a lot about not being able to write to the database and seemingly
crashing because of it.

This nginx configuration effectively load balances the Plex Transcoder across
multiple machines.

Environment Configuration
-------------------------

The master Plex server has read/write access to the database.  The slaves, on
the other hand, do not. The master server is configured like any normal Plex
server only the application data is hosted on an external share (could be local,
but you'd have to mount it from the slaves to access it). 

The slaves mount the external Plex application data as read only.  The exact 
flags that I used are:

    //nas/cache on /mnt/cache type cifs (ro,relatime,vers=1.0,cache=strict,domain=TOWER,uid=998,forceuid,gid=997,forcegid,addr=10.0.19.142,file_mode=0777,dir_mode=0777,nounix,serverino,rsize=61440,wsize=65536,actimeo=1)

Where uid and gid are the group and user id's of plex. You can determine that 
with:

    sudo -u plex id

I believe that the only necessary mount flags would be:

    ro, uid=998, gid=997, file_mode=0777, dir_mode=0777

Once the share is mounted, you have to symlink the appropriate files/folders to
the slaves Plex application data directory.  The symlinks are set up as follows:

![ScreenShot](http://i.imgur.com/9sRbK9A.png)

The Preferences.xml should be the same, but should be copied from the r/o
directory.  This allows you to modify configs on the slaves, but still maintain
the same Plex "server ID" that is used by plex.tv

The reason we get so granular with the symlinks is because we don't want to
symlink the "com.plexapp.plugins.library.db-shm" or 
"com.plexapp.plugins.library.db-shm" files. Those files are the temporary
database files that need to be created by the local Plex instance. If Plex can't
read or write to those files it will refuse to start. When Plex attempts to 
flush those files out to the main database, it throws errors in the log, but 
nothing mission critical as far as I can tell. 

Note: This whole thing is setup behind a NAT.  Plex thinks it's going to a 
single server when in reality it is being managed by nginx.  I currently have 
everything come in as follows:

Internet --:32400--> Router --:80--> nginx --:32400--> appropriate Plex instance 

Issues
------

There are still some weird issues with files not wanting to play.  This is
something I believe can be remedied by a more granular nginx config file. I'm
not versed enough in nginx to really determine what needs to be load balanced.
That's why I'm posting it up here :)
