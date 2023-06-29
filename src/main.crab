use std::env;
use std::fs::{ self, ReadDir };
use std::path::{ Path, PathBuf };

fn parse_cli () -> Result< ( PathBuf, PathBuf ), String > {

    let args: Vec<String> = env::args().collect();

    if args.len() != 3 {
        return Err( "The only allowed parameters are SOURCE and DEST.".to_string() )
    }

    let mut source: PathBuf = PathBuf::from( &args[1].to_string() );
    let mut destination: PathBuf = PathBuf::from( &args[2].to_string() );

    if source.is_relative() {
        source = env::current_dir().expect( "Failed to get Current Working Directory." ).join( source );
    }

    if destination.is_relative() {
        destination = env::current_dir().expect( "Failed to get Current Working Directory." ).join( destination );
    }

    return Ok(( source, destination ))

}

fn print_help () {

    let help_string: &str = "

fast-copy-rs

Copy files recursively from SOURCE to DEST as fast as possible. Preserves directory structure and traverses symlinks.

Syntax
      fast-copy-rs SOURCE DEST";

    println!( "{}", help_string );

}

fn get_source_paths_recursive<'a> ( dir: &Path, source_paths: &'a mut Vec<PathBuf> ) {

    let entries: ReadDir = match fs::read_dir( &dir ) {
        Ok ( entries ) => entries,
        Err ( error ) => { println!( "Unable to read contents of {}: {}", &dir.display().to_string(), error.kind() ); return },
    };

    let mime_types: Vec<&str> = Vec::from([ "aac", "abw", "arc", "avif", "avi", "azw", "bin", "bmp", "bz", "bz2", "cda", "csh", "css", "csv", "doc", "docx", "eot", "epub", "gz", "gif", "htm", "ico", "ics", "jar", "jpeg", "js", "json", "jsonld", "mid", "mjs", "mp3", "mp4", "mpeg", "mpkg", "odp", "ods", "odt", "oga", "ogv", "ogx", "opus", "otf", "png", "pdf", "php", "ppt", "pptx", "rar", "rtf", "sh", "svg", "tar", "tif", "ts", "ttf", "txt", "vsd", "wav", "weba", "webm", "webp", "woff", "woff2", "xhtml", "xls", "xlsx", "xml", "xul", "zip", "3gp", "3g2", "7z" ]);

    for entry in entries {

        let entry_path: PathBuf = entry.unwrap().path();

        if entry_path.is_file() {

            match &entry_path.extension() {
                None => { continue },
                Some ( extension )  => {

                    let extension_string: &str = match extension.to_str() {
                        None => { continue },
                        Some ( extension_string ) => extension_string,
                    };

                    if !mime_types.contains( &extension_string ) {
                        continue
                    }

                },
            }

            source_paths.push( entry_path );

        } else if entry_path.is_dir() {

            get_source_paths_recursive( &entry_path, source_paths );

        }

    }

}

fn copy_files ( source: &Path, destination: &Path, source_paths: &mut Vec<PathBuf> ) -> usize {

    let mut success_count: usize = 0;

    match fs::create_dir_all(  &destination ) {
        Ok (()) => (),
        Err ( error ) => { println!( "Unable to create {}: {}", destination.display().to_string(), error.kind() ); return success_count },
    }

    for source_path in source_paths {

        let destination_path: PathBuf = destination.join( source_path.strip_prefix( source ).unwrap() );

        if *source_path == destination_path {
            println!( "Unable to copy {}: SOURCE is DEST", source_path.display().to_string() );
            continue
        }

        match fs::create_dir_all( &destination_path.parent().unwrap() ) {
            Ok (()) => (),
            Err ( error ) => { println!( "Unable to create {}: {}", &destination_path.display().to_string(), error.kind() ); continue },
        }

        match fs::copy( &source_path, &destination_path ) {
            Ok ( _bytes_copied ) => { success_count += 1; },
            Err ( error ) => { println!( "Unable to copy {}: {}", source_path.display().to_string(), error.kind() ); continue },
        }

    }

    return success_count

}

fn main() {

    println!( "\nStarting fast-copy-rs" );
    
    let ( source, destination ): ( PathBuf, PathBuf ) = match parse_cli() {
        Ok (( s, d )) => ( s, d ),
        Err( err ) => { print_help(); println!( "{}", err ); return },
    };

    let mut source_paths: Vec<PathBuf> = Vec::new();
    get_source_paths_recursive( &source, &mut source_paths );

    let success_count: usize = copy_files ( &source, &destination, &mut source_paths );

    println!( "Copied {} of {} files.\n", success_count, source_paths.len() );

}