# Logs with geo
Ingest and enrich logs with geoip information (country, city, latitude, longitude).

![](hero.png)

The original idea here was to enrich my [apache](https://httpd.apache.org/) logs for [Loki](https://grafana.com/oss/loki/) so that I could use the Geomap view for my logs in [Grafana](https://grafana.com/).

## Setup on the server
1. Get the GeoLite-City.mmdb from MaxMind GeoIP and store it to the `geoip-db` folder.
2. Install requirements as root `python3 -m pip install -r requirements.txt` because we'll run the crontab as root later for accessing apache logs

### first run
Prepare two sample log files: sample-logs/access.log and sample-logs/other_vhosts_access.log

Attempt a first run like this

	python3 main.py -o sample-logs/all_with_geoip.log -f sample-logs/access.log -f sample-logs/other_vhosts_access.log

### cron
add this to roots crontab:

	* * * * * /usr/bin/python3 /YOUR/PATH/logs-with-geo/main.py -o /var/log/apache2/all_with_geoip.log -f /var/log/apache2/access.log -f /var/log/apache2/other_vhosts_access.log

To run it more often than once a minute (eg every 10 seconds), you could use this:

	*/1 * * * * /YOUR/PATH/logs-with-geo/runEvery.sh 5 "/usr/bin/python3 /YOUR/PATH/logs-with-geo/main.py -o /var/log/apache2/all_with_geoip.log -f /var/log/apache2/access.log -f /var/log/apache2/other_vhosts_access.log"

Watch all_with_geoip.log grow with added geo data (eg with `tail -f all_with_geoip.log`!

## Simulate behaviour (manual testing)
prepare these files:
- other_vhosts_access.log <- should be empty
- other_vhosts_access.orig.log <- should contain some log lines (at least 100 for more fun)

run this in one window:

	./log_faker.sh other_vhosts_access.orig.log other_vhosts_access.log

and this in another

	while [ 1 ]; do
        python3 main.py -o sample-logs/all_with_geoip.log -f sample-logs/access.log -f sample-logs/other_vhosts_access.log
	    sleep 3
	done

Then watch the sample-logs/all_with_geoip.log grow with added geo data!

## To Dos
- lint with flake8
- set up integration tests
- make it log source agnostic (not only apache, but also nginx or even something else that logs ips)
