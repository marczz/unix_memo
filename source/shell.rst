Shell Memo
==========

Reading query string
--------------------
To define variable ``a`` and ``b`` with respective values from the
``QUERY`` ``a=1234&b=9876``

.. code-block:: bash

   oldifs=$IFS
   IFS="&"
   set -- $QUERY
   IFS=$oldifs
   for l in $@; do
       eval $l
   done

Or in bash 4+ to define an associative array var with each value:

.. code-block:: bash

   declare -A var
   oldifs=$IFS
   IFS="&"
   set -- $QUERY
   IFS=$oldifs
   for i in $@; do
       IFS="="
       set -- $i
       IFS=$oldifs
       var[$1]=$2
   done


A more complete function that does uudecode is given in
`Parsing Web Form Input in CGI Shell Scripts
<http://cfajohnson.com/shell/articles/parse-query/>`_

.. code-block:: bash

   parse_query() #@ USAGE: parse_query var ...
   {
       local var val
       local IFS='&'
       vars="&$*&"
       [ "$REQUEST_METHOD" = "POST" ] && read QUERY_STRING
       set -f
       for item in $QUERY_STRING
       do
         var=${item%%=*}
         val=${item#*=}
         val=${val//+/ }
         case $vars in
             *"&$var&"* )
                 case $val in
                     *%[0-9a-fA-F][0-9a-fA-F]*)
                          printf -v val "%b" "${val//\%/\\x}"
                          ;;
                 esac
                 ;;
         esac
       done
       set +f
   }
