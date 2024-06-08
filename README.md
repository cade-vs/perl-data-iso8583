# NAME

    ISO8583 parsing and construction of message data stream

# SYNOPSIS

    use Data::ISO8583;
    use Data::ISO8583::VISA;
    
    my $msg_hash_ref = parse_iso8583_fields( $byte_data, $VISA_MESSAGE_FIELDS );
    
    my ( $fmap_arr_ref, $skip ) = parse_iso8583_bitmap( $byte_data, $len, $one, $base );

    my $field_62_hr = parse_iso8583_fields( $msg_hash_ref->{ 62 }, $VISA_MESSAGE_FIELD_62 );

    # field 61 has no bitmap, has only 3 fields, so to use the same parser,
    # fake bitmap has to be passed with first 3 bits set and also bitmap size
    # should be set to 1 (byte, BML) and flag raised that there is no chained bitmaps (BMO)
    my $field_61_hr = parse_iso8583_fields( chr( 0b11100000 ) . $msg_hash_ref->{ 61 }, $VISA_MESSAGE_FIELD_61, { BML => 1, BMO => 1 } );
    
    # field 60 is similar to 61 but there are 10 fields so either larger bitmap is needed (2 bytes)
    # or can be chained two 1-byte bitmaps:
    my $field_60_hr = parse_iso8583_fields( chr( 0b11111111 ) . chr( 0b01110000 ) . $msg_hash_ref->{ 60 }, $VISA_MESSAGE_FIELD_60, { BML => 1 } );

    or:

    my $field_60_hr = parse_iso8583_fields( pack( 'C2', 0b11111111, 0b01110000 ) . $msg_hash_ref->{ 60 }, $VISA_MESSAGE_FIELD_60, { BML => 1, BMB => 1 } );

    or with single bitmap of size 2 bytes without chaining:

    my $field_60_hr = parse_iso8583_fields( pack( 'C2', 0b11111111, 0b11000000 ) . $msg_hash_ref->{ 60 }, $VISA_MESSAGE_FIELD_60, { BML => 2, BMO => 1 } );

# FUNCTIONS

## parse\_iso8583\_fields( $byte\_data, $msg\_dictionary, \%options );

This functions parses the incoming data stream, using the given message 
dictionary and returns hash reference with parsed fields, keyed by field 
number.

%options argument is optional and is used mostly to pass parameters to
parse_iso8583_bitmap() as:

    {
    BML => $bitmap_length,
    BMO => $true_if_single_bitmap_only,
    BMB => $field_base_index_ie_first_field_number,
    }

## parse\_iso8583\_bitmap( $byte\_data, $len, $one )

This is helper function, used by parse_iso8583_fields() byt can be useful
standalone so it is exported. It takes data stream looks for primary and 
extended bitmaps (no limit for chained bitmaps) and returns array with 
found fields' numbers. It also returns how many bytes are read from the
incoming byte data for the bitmaps. The second return value (skip) is used
to skip bitmap data in the source data:

    my ( $fields_arr_ref, $skip ) = parse_iso8583_bitmap( $byte_data );
    my $fields_data = substr( $byte_data, $skip );
    
This function has two optional arguments:

    $len   -- size of the bitmaps in bytes, defaults to 8
    $one   -- if TRUE no chained bitmaps are searched and bit 1 is regular field
    $base  -- start index of the fields numbering
                * defaults to 2 no bitmap chaining
                * defaults to 1 if single bitmap
                * if specified, given number will be used regardless bitmap count

# TODO

    (more docs)

# DATA::ISO8583 SUB-MODULES

Data::ISO8583 package includes:

  * Data::ISO8583::VISA  -- VISA-specific dictionaries and functions

# GITHUB REPOSITORY

    https://github.com/cade-vs/perl-data-iso8583
    
    git@github.com:cade-vs/perl-data-iso8583.git
    
    git clone git@github.com:cade-vs/perl-data-iso8583.git
    
    or
    
    git clone https://github.com/cade-vs/perl-data-iso8583.git
    

# AUTHOR

    Vladi Belperchinov-Shabanski "Cade"
          <cade@noxrun.com> <cade@bis.bg> <cade@cpan.org>
    http://cade.noxrun.com/  

