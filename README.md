# Simple DOS Software Floppy Install

This is a guide on a very simplistic set of methods for creating installation 
disks for copying files to a DOS system. This does not use any traditional 
installation methods and instead uses either bare files or a ZIP to hold the
contents to be installed. This does cover modifying PATH, AUTOEXEC.BAT, and 
CONFIG.SYS for the system to use the new software.

## Why?

Getting raw files into a VM or onto real hardware is annoying. It's trivial
to build a disk image using [WinImage](https://www.winimage.com/winimage.htm)
or [mtools](https://www.gnu.org/software/mtools/) but then getting the files
into the right place on an DOS system is a chore. This aims to make it easy
to throw together a quick script to "install" files for custom disk images
to make it easier to build your own disks with your preferred programs ready
to go.

## File Transfers

There are two ways here that you could retrieve files from a
disk. My preference is to use direct when possible because it allows files
to be run from the disk still despite being more complicated.
  
## Direct

    REM Copy files
    XCOPY /E *.* %INSTALL_DIR%\
    
    REM Clean copied files
    del %INSTALL_DIR%\INSTALL.BAT
    del %INSTALL_DIR%\SED.COM
    del %INSTALL_DIR%\UNZIP.EXE
    del %INSTALL_DIR%\INFOZIP.LIC
    
This copies all files from the disk and then removes unneeded files copied
because there is no **exclude** available on MS-DOS 6.22.

## UnZip

    REM Decompress Files
    UNZIP %SOURCE_ZIP% -d %INSTALL_DIR%\

This unzips files using [Info-ZIP](https://infozip.sourceforge.net/) to the 
destination directory. 

## Examples

Three examples of different kinds of installations are provided to cover 
different software types.  

### PATH Programs

Full Install BAT File

    @ECHO OFF
    
    SET INSTALL_DIR=C:\BIN\VIM
    SET INSTALL_SED=C:\\BIN\\VIM
    
    echo Installing Vim to %INSTALL_DIR%
    
    REM Make install directory
    IF NOT EXIST %INSTALL_DIR%\NUL MKDIR /S %INSTALL_DIR%
    
    REM Copy files
    XCOPY /E *.* %INSTALL_DIR%\
    
    REM Clean copied files
    del %INSTALL_DIR%\INSTALL.BAT
    del %INSTALL_DIR%\SED.COM
    
    REM Add to PATH
    find /c /I "%INSTALL_DIR%" C:\AUTOEXEC.BAT >NUL
    if %ERRORLEVEL%==1 (
        IF EXIST C:\AUTOEXEC.IST DEL C:\AUTOEXEC.IST
        REN C:\AUTOEXEC.BAT C:\AUTOEXEC.IST
        SED "s/^PATH.*/$0;%INSTALL_SED%/" C:\AUTOEXEC.IST > C:\AUTOEXEC.BAT
    )

This is an example of installing a subset of files for [Vim 5.0 for MS-DOS](https://archive.org/details/vim50-msdos)
I would copy `vim.exe` and `xxd.exe` to the disk and nothing else. This would 
give you a minimal installation with hex functionality. But you could more more
files like help documents on the disk as well with this.

In order to modify the `PATH` environment variable a copy of [SED](https://www.pement.org/sed/)
MS-DOS is used. This allows us to find and modify the `PATH` line in 
`AUTOEXEC.BAT`, however it does now allow writing back to the same file. So a
backup file is used making it a two step process. Before any of that though,
`FIND` is used to determine if the destination directory is already likely
in the `PATH`. This check is imperfect however and may have false positives.

Also note that a second variable, `INSTALL_SED`, is created with *escaped*
back slashes for use with `SED`. There is no string substitute functionality
in base MS-DOS 6.22 so you must fix this manually or use a tool or shell like 
`4DOS` which can add this. The included method is the most compatible but is 
annoying to have to do.

## AUTOEXEC.BAT Launched Programs

Full Install BAT File
    
    @ECHO OFF
    
    SET INSTALL_DIR=C:\BIN
    
    SET AUTOEXEC_SYS=C:\BIN\CTMOUSE.EXE
    
    echo Installing Cute Mouse to %INSTALL_DIR%
    
    REM Make install directory
    IF NOT EXIST %INSTALL_DIR%\NUL MKDIR /S %INSTALL_DIR%
    
    REM Copy file
    xcopy ctmouse.exe C:\BIN
    
    REM Add to AUTOEXEC.BAT
    find /c /I "%AUTOEXEC_SYS%" C:\AUTOEXEC.BAT >NUL
    if %ERRORLEVEL%==1 echo %AUTOEXEC_SYS% >> C:\AUTOEXEC.BAT
    
This is an example of installing [Cute Mouse](https://cutemouse.sourceforge.net/) 
which only needs a single EXE file to be run from `AUTOEXEC.BAT` to work. The
main thing to note is that you could replace the variable for what write to
`AUTOEXEC.BAT` with multiple hard coded lines instead if you were adding 
something more complext. And that this can only postpend to the end of the file.
If you are using [MENUs](https://archive.org/details/msdos_manual_622/page/n29/mode/2up)
or have something else that must be run after, you will need to move the line 
to the appropriate location.


## CONFIG.SYS and AUTOEXEC.BAT Software

Full Install BAT File

    @ECHO OFF
    
    SET INSTALL_DIR=C:\BIN\DEST
    
    SET CONFIG_SYS=DEVICE=%INSTALL_DIR%\OAKCDROM.SYS /D:OAKCD
    SET AUTOEXEC_SYS=C:\DOS\MSCDEX.EXE /D:OAKCD
    
    echo Installing Oak CD-DROM Driver to %INSTALL_DIR%
    
    REM Make install directory
    IF NOT EXIST %INSTALL_DIR%\NUL MKDIR /S %INSTALL_DIR%
    
    REM Copy files
    XCOPY /E OAKCDROM.SYS %INSTALL_DIR%\
    
    REM Add to CONFIG.SYS
    find /c /I "%CONFIG_SYS%" C:\CONFIG.SYS >NUL
    if %ERRORLEVEL%==1 echo %CONFIG_SYS% >> C:\CONFIG.SYS
    
    REM Add to AUTOEXEC.BAT
    find /c /I "%AUTOEXEC_SYS%" C:\AUTOEXEC.BAT >NUL
    if %ERRORLEVEL%==1 echo %AUTOEXEC_SYS% >> C:\AUTOEXEC.BAT

This example installs the CD-ROM driver available on the Windows 98 Startup 
disk to an MS-DOS system. This is mostly the same as the **AUTOEXEC.BAT 
Launched Programs** script but includes an additional block for the `CONFIG.SYS`
file.
