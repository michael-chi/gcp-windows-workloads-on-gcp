```bash

ACCESS_TOKEN=`gcloud auth print-access-token`
export SERVICE_ID=custom:demo-movie-web
export SLO_ID=uptime_slo
export CREATE_SLO_POST_BODY=$(cat <<EOF
{
   "displayName": "Website Availability",
   "serviceLevelIndicator": {
      "requestBased": 
        {
            "distributionCut":{
            "distributionFilter": "metric.type=\"loadbalancing.googleapis.com/https/total_latencies\" resource.type=\"https_lb_rule\"",
            "range": 
                {
                    "min": 0,
                    "max": 5000
                }
            }
        }
   },
   "goal": 0.99,
   "rollingPeriod": "86400s",
   "displayName": "Latency SLO"
}
EOF
)
#   Create a custom service
curl  --http1.1 --header "Authorization: Bearer ${ACCESS_TOKEN}" --header "Content-Type: application/json" -X POST -d "${CREATE_SERVICE_POST_BODY}" https://monitoring.googleapis.com/v3/projects/${PROJECT_ID}/services?service_id=${SERVICE_ID}

#   Create an SLO
curl  --http1.1 --header "Authorization: Bearer ${ACCESS_TOKEN}" --header "Content-Type: application/json" -X POST -d "${CREATE_SLO_POST_BODY}" https://monitoring.googleapis.com/v3/projects/${PROJECT_ID}/services/${SERVICE_ID}/serviceLevelObjectives?service_level_objective_id=${SLO_ID}
```