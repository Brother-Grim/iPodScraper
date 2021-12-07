# -*- coding: utf-8 -*-
"""Flatten iPod folder/file structure into a new folder."""
import os
import shutil
import re
import time
import logging
import mutagen  # pip install mutagen


####
# User Inputs
# Directory to look for files in. No trailing forward slash.
inputDir = '/home/eric/Documents/code/python/pyPodScraper/Input'
# Directory to move files to. No trailing slash.
outputDir = '/home/eric/Documents/code/python/pyPodScraper/Output'
# Extension(s) to look for.
extList = ['.mp3', '.m4a', '.mp4']
# If you'd like to create an artist folder and place related songs in it
makeArtist = True
# End User Inputs
###

# Logging stuff
logging.basicConfig(level=logging.DEBUG,
                    format='%(asctime)s - %(name)s -' +
                    '%(levelname)s - %(message)s',
                    datefmt='%m-%d %H:%M',
                    filename='log.log',
                    filemode='w')
mutagenLog = logging.getLogger('MutagenError')
moveFileLog = logging.getLogger('MoveFileError')


def timing(f):
    """Get function timings."""
    def wrap(*args):
        time1 = time.time()
        ret = f(*args)
        time2 = time.time()
        print '%s function took %0.3f ms' % (f.func_name, (time2-time1)*1000.0)
        return ret
    return wrap


@timing
def walk_directory():
    """Walk over a given `inputDir` directory's files and move to new location.
    Files are compared to extension list and if a match is found the file is
    renamed according to mutagen metadata title tags (if available) and moved
    to a new `outputDir` in a flat data structure or by artist.
    """
    for (root, folders, files) in os.walk(inputDir):
        for filename in files:
            if does_extension_match(filename):
                # print 'extension match:', does_extension_match(filename)
                make_output_subfolder(outputDir)
                # print 'output subfolder exists:', os.path.exists(outputDir)
                oldFilepath = os.path.join(root, filename)
                # print 'oldFilepath:', oldFilepath
                newFilepath = get_new_path(oldFilepath, filename, makeArtist)
                # print 'newFilepath:', newFilepath
                if newFilepath:
                    move_file(oldFilepath, newFilepath)
                    verify_file_move(newFilepath, filename)


def does_extension_match(filename):
    """Compare filename's extension to global extList."""
    return os.path.splitext(filename)[1].lower() in extList


def get_new_path(filepath, filename, useArtistFolder):
    """Return a path string when a filepath is given.
    Using mutagen the audio file's tags will be accessed and if a 'title' tag
    can be found it will be used as the new title.
    If no 'title' tag can be found then the original file name will be used and
    the file will be placed in the specified `missingDir` directory.
    """
    try:
        fileTags = mutagen.File(filepath, easy=True)
    except:
        print 'BOOP'
        mutagenLog.warning(str(filepath))
        return

    ext = os.path.splitext(filename)[1]
    # Get title
    if 'title' in fileTags:
        title = fileTags['title'][0].encode('utf-8') + ext.lower()
        # Remove illegal characters from title
        title = re.sub(r'[/\\:*?"<>|]', '', title)
    else:
        missingDir = os.path.join(outputDir, 'UnknownTitle')
        make_output_subfolder(missingDir)
        newPath = os.path.join(missingDir, filename)
        return newPath

    # Get artist
    # TODO refactor this
    if useArtistFolder:
        if 'artist' in fileTags:
            artist = fileTags['artist'][0].encode('utf-8')
            # Remove illegal characters from title
            artist = re.sub(r'[/\\:*?"<>|]', '', artist)
            make_output_subfolder(os.path.join(outputDir, artist))
        else:
            artist = 'UnknownArtist'
            make_output_subfolder(os.path.join(outputDir, 'UnknownArtist'))

        newPath = os.path.join(outputDir, artist, title)
        return newPath
    newPath = '%s/%s' % (outputDir, title)
    return newPath


def make_output_subfolder(path):
    """Given a path check if exists and if missing then create."""
    # Note: there is a race-condition with this method of directory check
    # and create. This will not apply here, but if by some chance the path were
    # to be created between `if not os.path...` and `os.makedirs...` then an
    # error would be raised.
    if not os.path.exists(path):
        os.makedirs(path)


def move_file(oldPath, newPath):
    """Move file at old path to the new path."""
    logging.basicConfig(filename='failedToMove.log', level=logging.DEBUG)
    if os.path.isfile(oldPath) and os.path.exists(os.path.dirname(newPath)):
        try:
            shutil.copy2(oldPath, newPath)
        except:
            moveFileLog.info('Not moved: ' + oldPath)
        return
    else:
        if not os.path.isfile(oldPath):
            moveFileLog.warning('The original filepath is ' +
                                'no longer valid :: ' + oldPath)
            raise Exception('The original filepath is no longer valid :: ' +
                            oldPath)
        else:
            moveFileLog.warning('The new directory path is not valid :: ' +
                                os.path.dirname(newPath))
            raise Exception('The new directory path is not valid :: ' +
                            os.path.dirname(newPath))


def verify_file_move(path, filename):
    """Given a file print the filename and a symbol denoting its presence."""
    if os.path.exists(path):
        print '\t %s ✓' % (filename)
    else:
        print '\t %s x' % (filename)


walk_directory()
