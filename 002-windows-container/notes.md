Invoke-WebRequest -Headers @{"Metadata-Flavor"="Google"} -Method 'GET' -Uri 'http://metadata.google.internal/computeMetadata/v1/instance/scheduling?recursive=true' | select Content | ft

