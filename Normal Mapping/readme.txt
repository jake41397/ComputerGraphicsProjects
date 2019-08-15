The Shader code is used in defining the material on lines 113 and 114.

VERTEX SHADER CODE

<script id="vertexShader" type="x-shader/x-vertex">
    precision highp float;

    attribute vec3 position;
    attribute vec3 normal;
		attribute vec2 uv;

    uniform mat4 modelViewMatrix;
		uniform mat4 viewMatrix;
    uniform mat4 projectionMatrix;
    uniform mat3 normalMatrix; // inverse transpose of modelViewMatrix

    varying vec2 vUV;
		varying vec3 vNormal;
		varying vec3 vPosition;

    void main()
		{
      vUV = uv;
			vNormal = normalize(normalMatrix * normal);
			vPosition = (modelViewMatrix*vec4(position, 1.0)).xyz;
      gl_Position = projectionMatrix*modelViewMatrix*vec4(position, 1.0 );
    }
  </script>


FRAGMENT SHADER CODE

  <script id="fragmentShader" type="x-shader/x-fragment">
    precision highp float;

		uniform sampler2D tNormal;
		uniform sampler2D image;
		uniform mat4 viewMatrix;
		uniform vec3 lightPosition;

		varying vec2 vUV;
		varying vec3 vNormal;
		varying vec3 vPosition;
		varying vec3 vertexNormal;
		varying vec3 vertexPosition;

    void main()
		{
			// Normal Mapping calculation
			vec3 Q0 = dFdx(vPosition);
			vec3 Q1 = dFdy(vPosition);
			vec2 st0 = dFdx(vUV);
			vec2 st1 = dFdy(vUV);
			float denom = st1.t*st0.s - st0.t*st1.s;
			float denomSign = sign(denom);
			vec3 T = normalize((Q0*st1.t - Q1*st0.t)*denomSign);
			vec3 B = normalize((-Q0*st1.s + Q1*st0.s)*denomSign);
			vec3 N = normalize(vNormal);
			vec3 mapN = texture2D(tNormal, vUV).xyz * 2.0 - 1.0;
			mapN.xy *= (float(gl_FrontFacing)*2.0-1.0);
			vec3 newN =  normalize(mat3(T,B,N)*mapN);

			// Do Lambertian lighting computation
			vec3 L = normalize((viewMatrix * vec4(lightPosition, 1.0)).xyz - vPosition);
			float NdotL = dot(newN, L);

			// Clamp NdotL to (0,1)
			if (NdotL > 1.0)
			{
				NdotL = 1.0;
			}
			else if (NdotL < 0.0)
			{
				NdotL = 0.0;
			}

			gl_FragColor = texture2D(image, vUV) * NdotL;
    }
  </script>