
Ever used psm1 file to write reusable common functions through other ps1 files. 

Had the horrible experience after editing the psm1 files, the updates were not reflected because of the module was being cached. It took a while to figure it while trying different string concatenations options from Join, String interpolation, to using the + sign. -f operator is used to format strings 

The only was to load the new psm1 was to kill the powershell session and start a new one. 