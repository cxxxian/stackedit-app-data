```glsl
void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    // ========================
    // Camera setup
    // ========================
    vec3 cameraPos = vec3(0.0, 2.0, 2.0);
    vec3 cameraTarget = vec3(0.0, 0.0, -2.0);
    vec3 cameraDir = normalize(cameraTarget - cameraPos);
    vec3 cameraUp  = vec3(0.0, 1.0, 0.0);
    vec3 cameraRight = normalize(cross(cameraDir, cameraUp));
    cameraUp = cross(cameraRight, cameraDir);

    // Screen coordinates
    vec2 uv = (fragCoord.xy / iResolution.xy) * 2.0 - 1.0;
    uv.x *= iResolution.x / iResolution.y;

    // Ray direction
    vec3 rayDir = normalize(cameraDir + uv.x*cameraRight + uv.y*cameraUp);

    // ========================
    // Sphere definition
    // ========================
    vec3 sphereCenter = vec3(0.0, 0.0, -2.0);
    float sphereRadius = 1.0;

    // Ray-sphere intersection
    vec3 oc = cameraPos - sphereCenter;
    float b = dot(oc, rayDir);
    float c = dot(oc, oc) - sphereRadius*sphereRadius;
    float h = b*b - c;

    vec3 color = vec3(0.0);

    if(h > 0.0)
    {
        float t = -b - sqrt(h); // take the nearest intersection
        vec3 hitPos = cameraPos + t*rayDir;
        vec3 normal = normalize(hitPos - sphereCenter);

        // ========================
        // Phong lighting
        // ========================
        vec3 lightPos = vec3(-1.0, -1.0, -1.0);
        vec3 lightDir = normalize(lightPos - hitPos);
        vec3 viewDir  = normalize(cameraPos - hitPos);

        float ambient = 0.1;
        float diffuse = max(dot(normal, lightDir), 0.0);
        float specular = 0.0;
        if(diffuse > 0.0)
        {
            vec3 reflectDir = reflect(-lightDir, normal);
            specular = pow(max(dot(viewDir, reflectDir), 0.0), 32.0);
        }

        vec3 lightColor = vec3(1.0, 0.0, 0.0); // red light
        color = (ambient + diffuse + specular) * lightColor;
    }

    fragColor = vec4(color, 1.0);
}

```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzMjIyMTAwMTZdfQ==
-->