puts "cdr_recovery.tcl script"
 
 # user vars
 # the folder to try to copy files from
 set sourcedir "/media/cdrom"
 # the folder to try to copy them to
 set targetdir "/home/andy/980720_1616_linux_s1"
 # the number of time to re-try
 set numits 100
 
 # util vars
 set count 0
 set totalsize 0

 # create the list of files from a big glob
 set sourcefilelist [ glob -nocomplain -directory $sourcedir  \
 * \
 */* \
 */*/* \
 */*/*/* \
 */*/*/*/* \
 */*/*/*/*/* \
 */*/*/*/*/*/* \
 */*/*/*/*/*/*/* \
 ]

 if  { [ llength $sourcefilelist ] == 0 } {
     puts "no matches"
     exit
 }

 # loop through the file list building up an array with all the info in
 foreach sourcefile $sourcefilelist {
     incr count 
     # build up the data array
     set filearray(sfile,$count) "$sourcefile"
     set filearray(size,$count) "[file size $sourcefile]"
     set filearray(date,$count) "[clock format [file atime $sourcefile] -format "%H:%M:%S %D"]"
     set filearray(trimfile,$count) "[string trimleft $sourcefile "$sourcedir"]"
     set filearray(fileonly,$count) "[file tail $sourcefile]"  
     set filearray(folderonly,$count) "[string trimleft  [file dirname $filearray(sfile,$count)] "$sourcedir" ]"
     set filearray(done,$count) 0

     # print out some general info
     puts "$filearray(sfile,$count)"
     puts "  number - $count"
     puts "  size - $filearray(size,$count) bytes"
     puts "  date - $filearray(date,$count)"
     puts "  trim - $filearray(trimfile,$count)"
     puts "  file - $filearray(fileonly,$count)"
     puts "  dir  - $filearray(folderonly,$count)"
    
     # keep a running total of all the file's sizes
     set totalsize [ expr $totalsize + $filearray(size,$count) ]
 }

 # save the number of files
 set numfiles $count
 # print some useful sumary info
 puts " "
 puts "total size: $totalsize bytes"
 puts "file count: $numfiles"
 puts " "

 # do them all numits num of times
 for { set it 1 } { $it <= $numits } { incr it } { 
     # loop through the array
     for { set count 1 } { $count <= $numfiles } { incr count } { 
         # if the file is not done already
         if { $filearray(done,$count) == 0 } {
             puts "$filearray(sfile,$count)"
             puts "  number - $count"
             puts "  size - $filearray(size,$count) bytes"
             puts "  date - $filearray(date,$count)"
             puts "  trim - $filearray(trimfile,$count)"
             puts "  file - $filearray(fileonly,$count)"
             puts "  dir  - $filearray(folderonly,$count)"

             # see if the target dir exists
             if { [ file isdirectory $targetdir/$filearray(folderonly,$count) ] == 1 } {
                 puts "  exists: $targetdir/$filearray(folderonly,$count) "
             } else {  
                 puts "  doesnt exist: $targetdir/$filearray(folderonly,$count) "
                 file mkdir "$targetdir/$filearray(folderonly,$count)"
                 puts "  created it"
             } 
    
             puts "  about to copy:"
             puts "    $filearray(sfile,$count) "
             puts "  to"
             puts "    $targetdir/$filearray(folderonly,$count)"
             if { [ file exists $targetdir/$filearray(trimfile,$count) ] == 1 } {
                 # the file already exists
                 puts "  file already exixts, no need to copy"
                 set filearray(done,$count) 1
             } elseif { [ catch { file copy -force $filearray(sfile,$count) $targetdir/$filearray(folderonly,$count)  } errorvar ] } {
                 # catch an error if there is one, and print details
                 puts "  error: $errorvar"
             } else {
                 # if there is no error, say so, and set done to true
                 puts "  no error, copied ok"
                 set filearray(done,$count) 1
             }
             puts " "
         }
     }
     puts "it $it"
     puts " "
 }
 
 puts "script complete"
