The Shader code is used in defining the material on lines 102 and 103.

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
		varying vec3 vertexNormal;
		varying vec3 vertexPosition;

    void main()
		{
      vUV = uv;
			vNormal = normalize(normalMatrix * normal);
			vPosition = (modelViewMatrix*vec4(position, 1.0)).xyz;
      gl_Position = projectionMatrix*modelViewMatrix*vec4(position, 1.0 );
			vertexPosition = (modelViewMatrix*vec4(position, 1.0)).xyz;
			vertexNormal = normal;
    }
  </script>

// FRAGMENT SHADER CODE

  <script id="fragmentShader" type="x-shader/x-fragment">
    precision highp float;

		uniform samplerCube cubemap;
		uniform vec3 lightPosition;
		uniform mat4 viewMatrix;
		uniform vec3 cameraPosition;

		varying vec2 vUV;
		varying vec3 vNormal;
		varying vec3 vPosition;
		varying vec3 vertexNormal;
		varying vec3 vertexPosition;

    void main()
		{
			// Do Lambertian lighting computation
			//vec3 N = normalize((viewMatrix * vec4(vertexNormal, 1.0)).xyz);
			vec3 N = normalize(vNormal);
			vec3 L = normalize((viewMatrix * vec4(lightPosition, 1.0)).xyz - vPosition);
			float NdotL = clamp(dot(N,L), 0.5, 1.5);

			// Environment Mapping Code
			vec3 I = normalize(vertexPosition - (viewMatrix * vec4(cameraPosition, 1.0)).xyz);
			vec3 R = reflect(I, N);
			mat3 vM = mat3(viewMatrix);
			vec3 R_in_worldspace = vec3(dot(vM[0],R),dot(vM[1],R),dot(vM[2],R));
			vec4 envMapColor = textureCube(cubemap, R_in_worldspace);

			gl_FragColor = envMapColor * NdotL;
    }
  </script>