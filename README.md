# CacheManager
Provide a caching mechanism to files in any powershell script.

# How to use
```powershell
#Imports the module!
import-module CacheManager;

#Creates a new cache manager object!
$MyCache = New-CacheManager;

#Specify a cache manager directory
$MyCache.cacheDirectory = 'C:\MyCache'

#Initialize the cache manager!
#At this point cache manager will prepare the cache directory and your internal database.
#If the cache directory was previously used to create a cache, it will load the database.
#This allows cache manager persist all cache information between powershell sessions using same directory.
$MyCache.init()

#Start caching your remote files!
$PathToMyFile = $MyCache.getFile('\\MyRemoteServer\MyDir\MyFile.txt');

#The cache manager will manage the copy of remote file to local file!;
#When you update remote file, a next call to getFile will update local file!

#If you have a list of you remotes files, you can peridically update them!

#for example, imagine that you have a script that loads a lot of sql files from network to execute against a sql server instance!
#Imagine that you execute it in a loop...
$MyFilesOriginal = gci '\\MyRemote\MyFiles\*.sql' | %{$_.FullName};
$MyFiles  = @();

$execute = $true;
$LastCacheCheck = $null;

while($execute){

  #Every 5 minutes, update file cache just calling getFile...
  if( ((Get-Date)-$LastCacheCheck).totalMinutes -ge 5 -or !$LastCacheCheck){
          $MyFiles = @();
          $MyFilesOriginal | %{
              $MyFiles += $MyCache.getFile($_);
          }
          
          $LastCacheCheck = (Get-Date);
  }

  #Execute each file!
  $MyFiles | %{
       $CurrentFile = $_;
       $results = Invoke-SqlCmd -ServerInstance MyServer -Database mydatabase -InputFile $CurrentFile;
    }
    
    #Waits for a time...
    Start-Sleep -s 120;
}


```
