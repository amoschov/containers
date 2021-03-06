# based on https://github.com/dperson/smokeping
FROM bsantanna/ubuntu-multi:16.04
MAINTAINER Bruno Sant'Anna "bruno@btech.software"
ENV REFRESHED_AT 2017-10-25

# Install lighttpd and smokeping
RUN export DEBIAN_FRONTEND='noninteractive' && \
    apt-get update -qq && \
    apt-get install -qqy --no-install-recommends ca-certificates curl dnsutils \
                echoping fonts-dejavu-core lighttpd procps smokeping ssmtp \
                $(apt-get -s dist-upgrade|awk '/^Inst.*ecurity/ {print $2}') &&\
    apt-get clean

RUN echo '+ to = |/usr/bin/smokeping-alert\n' \
                >>/etc/smokeping/config.d/Alert && \
    echo '+ from = smokealert@localhost\n\n' \
                >>/etc/smokeping/config.d/Alert && \
    echo '+ hostdown\ntype = loss\npattern = >60%\nedgetrigger = yes\ncomment = Massive loss for 5 mins\n' \
                >>/etc/smokeping/config.d/Alert

RUN echo '+ EchoPingHttp\n\nbinary = /usr/bin/echoping\n' \
                >>/etc/smokeping/config.d/Probes && \
    echo '+ EchoPingHttps\n\nbinary = /usr/bin/echoping\n' \
                >>/etc/smokeping/config.d/Probes && \
    echo '+ Curl\n\nbinary = /usr/bin/curl\nstep = 60\nurlformat = http://%host%/\n' \
                >>/etc/smokeping/config.d/Probes

RUN sed -i '/^syslogfacility/s/^/#/' /etc/smokeping/config.d/General && \
    sed -i 's/the \(SmokePing website\) of xxx Company/our \1/' \
                /etc/smokeping/config.d/General

RUN conf=/etc/lighttpd/lighttpd.conf dir=/etc/lighttpd/conf-available \
                header=setenv.add-response-header && \
    sed -i '/server.errorlog/s|^|#|' $conf && \
    sed -i '/server.document-root/s|/html||' $conf && \
    sed -i '/mod_rewrite/a\ \t"mod_setenv",' $conf && \
    echo "\\n$header"' += ( "X-XSS-Protection" => "1; mode=block" )' >>$conf &&\
    echo "$header"' += ( "X-Content-Type-Options" => "nosniff" )' >>$conf && \
    echo "$header"' += ( "X-Robots-Tag" => "none" )' >>$conf&& \
    echo "$header"' += ( "X-Frame-Options" => "SAMEORIGIN" )' >>$conf && \
    echo '\n$HTTP["url"] =~ "^/smokeping($|/)" {' >>$conf && \
    echo '\tdir-listing.activate = "disable"\n}' >>$conf && \
    echo '\n# redirect to the right Smokeping URI' >>$conf && \
    echo 'url.redirect  = ("^/$" => "/smokeping/smokeping.cgi",' >>$conf && \
    echo '\t\t\t"^/smokeping/?$" => "/smokeping/smokeping.cgi")' >>$conf && \
    sed -i 's|var/log/lighttpd/access.log|tmp/log|' $dir/10-accesslog.conf && \
    sed -i '/^#cgi\.assign/,$s/^#//; /"\.pl"/i\ \t".cgi"  => "/usr/bin/perl",' \
                $dir/10-cgi.conf && \
    echo '\nfastcgi.server += ( ".cgi" =>\n\t((' >>$dir/10-fastcgi.conf && \
    sed -i -e '/CHILDREN/s/[0-9][0-9]*/16/' \
                -e '/max-procs/a\ \t\t"idle-timeout" => 20,' \
                $dir/15-fastcgi-php.conf && \
    grep -q 'allow-x-send-file' $dir/15-fastcgi-php.conf || { \
        sed -i '/idle-timeout/a\ \t\t"allow-x-send-file" => "enable",' \
                    $dir/15-fastcgi-php.conf && \
        sed -i '/"bin-environment"/a\ \t\t\t"MOD_X_SENDFILE2_ENABLED" => "1",' \
                    $dir/15-fastcgi-php.conf; } && \
    echo '\t\t"socket" => "/tmp/perl.socket" + var.PID,' \
                >>$dir/10-fastcgi.conf && \
    echo '\t\t"bin-path" => "/usr/lib/cgi-bin/smokeping.cgi",'\
                >>$dir/10-fastcgi.conf && \
    echo '\t\t"docroot" => "/var/www",' >>$dir/10-fastcgi.conf && \
    echo '\t\t"check-local"     => "disable",\n\t))\n)' \
                >>$dir/10-fastcgi.conf && \
    unset conf dir header

RUN lighttpd-enable-mod cgi && lighttpd-enable-mod fastcgi

RUN if [ ! -d /var/cache/smokeping ] ; then mkdir -p /var/cache/smokeping ; fi && \
    if [ ! -d /var/lib/smokeping ] ; then mkdir -p /var/lib/smokeping ; fi && \
    if [ ! -d /run/smokeping ] ; then mkdir -p /run/smokeping ; fi && \
    if [ -d /var/www/cgi-bin ] ; then rmdir /var/www/cgi-bin ; fi && \
    ln -s /usr/share/smokeping/www /var/www/smokeping && \
    ln -s /usr/lib/cgi-bin /var/www/ && \
    ln -s /usr/lib/cgi-bin/smokeping.cgi /var/www/smokeping/ && \
    chown -Rh smokeping:www-data /var/cache/smokeping /var/lib/smokeping /run/smokeping && \
    chmod -R g+ws /var/cache/smokeping /var/lib/smokeping /run/smokeping && \
    chmod u+s /usr/bin/fping && \
    rm -rf /var/lib/apt/lists/* /tmp/*

ADD entrypoint.sh /sbin/entrypoint
RUN chmod +x /sbin/entrypoint

ADD scripts/util/user_digest.sh /usr/bin/user_digest
RUN chmod +x /usr/bin/user_digest

ADD scripts/runtime/smokeping-alert.sh /usr/bin/smokeping-alert
RUN chmod +x /usr/bin/smokeping-alert

VOLUME ["/etc/smokeping", "/etc/ssmtp", "/var/lib/smokeping", "/var/cache/smokeping"]
EXPOSE 80

ENTRYPOINT ["/sbin/entrypoint"]