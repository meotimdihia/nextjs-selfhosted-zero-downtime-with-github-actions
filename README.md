# How to Deploy a Self-Hosted Next.js App on a Server with Near Zero Downtime

(Achieving true zero downtime is difficult—I still haven’t found a simple way to do it.)

While big corporations or large companies can hire a team to deploy Next.js using Docker, for a startup or individual developer with limited experience in Docker, this guide provides a simple method to deploy Next.js with near zero downtime.

⸻

# 1. Use PM2

I think you can use PM2 with Capistrano-like deployments to handle this problem. However, I haven’t tested it yet because PM2 monkey-patches logs, which is inappropriate in my case.

🔗 Reference: [Capistrano-like deployments with PM2](https://pm2.keymetrics.io/docs/tutorials/capistrano-like-deployments)

⸻

# 2. Use systemd

This method is very simple, but troubleshooting issues may be difficult if you lack experience.

I used this GitHub Actions workflow to deploy a full website serving 50,000 active users simultaneously.

How It Works:
	1.	Create multiple systemd services such as project.service, project2.service, etc.
	2.	The number of services depends on how large your website is—you will need more as your traffic grows.
	3.	When deploying a new version, swap the directory and restart services to ensure minimal downtime.

You can find more details in the full workflow file: `.github/workflows/main.yml`

``` shell
# Copy the .env file from the old directory to the new one
cp -R /home/meotimdihia/sources/project/.env /home/meotimdihia/sources/project3/.env

# Stop all instances
sudo systemctl stop project2 project3 project4

# Rename the old project directory as a backup
mv /home/meotimdihia/sources/project /home/meotimdihia/sources/project2

# Replace the old directory with the newly deployed version
cp -R /home/meotimdihia/sources/project3/. /home/meotimdihia/sources/project/

# Start all instances
sudo systemctl start project project2 project3 project4

# Since you are using a self-hosted setup, you need to clear the CDN cache
curl -X POST "https://api.cloudflare.com/client/v4/zones/2d75d17485a663129e45e83a853aede1/purge_cache" \
    -H "X-Auth-Email: youremail@gmail.com" \
    -H "Authorization: Bearer ${{ secrets.CLOUDFLARE_KEY }}" \
    -H "Content-Type: application/json" \
    --data '{"files":["https://yourdomain.io", "https://yourdomain.io/home?utm_source=web_app_manifest", "https://yourdomain.io/sw.js", "https://yourdomain.io/api/get-build-id"]}'

# Now you can delete the old version.
# Why do we delete it as the last step? Because this directory is very large when using self-hosting.
# In my case, it can grow up to 200GB.
rm -rf /home/meotimdihia/sources/project2
```
