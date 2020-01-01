# PHP – get domain name from URL

Getting the domain name from an URL (in PHP) seems to be a trivial task at first, but after further consideration you start to see the fine print. Because top level domains (TLDs) can have an arbitrary amount of words (for example there is .com, .co.uk., .lib.ca.us, and so on), it’s not possible to tell where the domain name ends and the TLD begins, without having a full list of all TLDs. Fortunately the Mozilla foundation has compiled and published such a list [here](https://publicsuffix.org/list/effective_tld_names.dat), under the [Mozilla Public License v 2.0](https://www.mozilla.org/en-US/MPL/2.0/). 

You can use the list together with this code to get the domain name from an URL:

```php
<?php
    function get_hostname_from_url($url)
    {
        /* 
        PHP function that gets the hostname (subdomain+domain) from an URL
        
        This functions extracts the subdomain + domain from an URL. 
        For example for the $url: “http://mail.google.com/example” it would return “mail.google.com”. 
        It does so by first removing the http protocol, and then considering the text up until the first “/” it encounters to be part of the hostname.

        Note that this function only works for URLs with the http and https protocol. 
        For other protocols, you can either use another str_replace function, or adapt it to a more general solution.
        */

        $url = str_replace('https://', '', $url);
        $url = str_replace('http://', '', $url);
        $domain = '';
        $i = 0;
        while ($i < strlen($url))
        {
            if ($url[$i] == '/') break;
            $domain .= $url[$i];
            $i++;
        }
        return $domain;
    }

    function get_domain_name_from_url($url) {
        try
        {
            $tld_list_filename_to_be_used = 'effective_tld_names.dat';
                        
            $fh = fopen($tld_list_filename_to_be_used, 'r');
            $domain_suffixes = array();
            if ($fh)
            {
                while (($line = fgets($fh)) !== false) 
                {
                    $trimmed_line = trim($line); // remove whitespace from the beginning and end of the line
                    if ($trimmed_line !== '') {
                        if (strpos($trimmed_line, '//') === FALSE) {
                            // if the line is not a comment
                            $domain_suffixes[$trimmed_line] = true;
                        }
                    }
                }
                fclose($fh);
            }
            
            $hostname = get_hostname_from_url($url);
            $domain = $hostname;
                
            // DEBUG that tests if the suffix list is properly initialized
            // print_r($domain_suffixes);
            
            // try to get domain name without subdomain
            // this can be done by finding the longest TLD suffix that fits
            $hostname_components = explode('.', $hostname);
            $count_hostname_components = count($hostname_components);
            for ($i = 0; $i < $count_hostname_components; $i++)
            {
                $potential_domain_suffix = '';
                $first = true;
                for ($j = ($i+1); $j < $count_hostname_components; $j++)
                {
                    if (!$first) $potential_domain_suffix .= '.';
                    $potential_domain_suffix .= $hostname_components[$j];
                    if ($first) $first = false;
                }
                if (isset($domain_suffixes[$potential_domain_suffix]))
                {
                    $domain = $hostname_components[$i].'.'.$potential_domain_suffix;
                    break;
                }
            }
            
            return $domain;
        }
        catch (Exception $e)
        {
            error_log($e -> getMessage());
            return '';
        }
    }
?>
```
