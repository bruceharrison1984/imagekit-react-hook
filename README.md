# imagekit-react-hook

[![npm](https://img.shields.io/npm/v/imagekit-react-hook?color=orange)](https://www.npmjs.com/package/imagekit-react-hook)

This is a simple hook wrapper around the ImageKit.io javascript library. It allows for easy usage of client-side uploads,
as well as generating signed URLs for private images.

This is a straight-forward, non-opinionated wrapper, so [the original
ImageKit.io documentation](https://github.com/imagekit-developer/imagekit-javascript) should be referred to for method usage.

## Usage

To use, simply wrap the components you wish to use the `useImageKit` hook with the included `ImageKitProvider`. You must provide the
`urlEndpoint`, `authenticationEndpoint`, and `publicKey` properties.

This example uses NextJS and loads the settings from environment variables, but any framework based on React should be similar:

```tsx
import { ImageKitProvider } from '@/providers/ImageKitProvider';

const PhotosPage = () => {
  const [origin, setOrigin] = useState<string>();

  /* our authentication end point is on the same server */
  useEffect(() => setOrigin(window.location.origin), []);

  return (
    <ImageKitProvider
      urlEndpoint={process.env.NEXT_PUBLIC_IMAGEKIT_URL_ENDPOINT!}
      authenticationEndpoint={`${origin}/api/auth/photo`}
      publicKey={process.env.NEXT_PUBLIC_IMAGEKIT_PUBLIC_KEY}
    >
      <child-components />
    </ImageKitProvider>
  );
};

export default PhotosPage;
```

Child components within `ImageKitProvider` can access the ImageKitContext via the `useImageKit` hook.

This example shows how you could upload a single profile image that a user selected via an `Input` element:

```tsx
import { v4 as uuidV4 } from 'uuid';
import { useImageKit } from 'imagekit-react-hook';

const { upload, url } = useImageKit();

/* attach to the onChange event of the input element */
const onChange = async (event: ChangeEvent<HTMLInputElement>) => {
  if (!event.target.files)
    throw new Error('File list is null after selecting a file!');

  const profileImage = event.target.files[0];
  const ext = profileImage.name.split('.').pop();

  await upload({
    file: profileImage,
    /* we generate a random filename for ImageKit. You can pass through the original filename if you like */
    fileName: [uuidV4(), ext].join('.'),
    extensions: [
      {
        name: 'google-auto-tagging',
        minConfidence: 90,
        maxTags: 2,
      },
    ],
  });
};
```

## Upload Method

The `upload` method is for client-side image uploads. This _requires_ the `authenticationEndpoint` property also
being set to an endpoint that conforms to the specification required.

It is up to you how and where you implement this endpoint, but ImageKit.io offers [this documentation](https://docs.imagekit.io/api-reference/upload-file-api/client-side-file-upload#signature-generation-for-client-side-file-upload).

## Hook Methods & Props

The following methods and properties are exposed via the `useImageKit` hook:

| Method/Prop            | Type   | Description                                                                |
| ---------------------- | ------ | -------------------------------------------------------------------------- |
| upload                 | method | Used for client-side uploading of images (requires authenticationEndpoint) |
| url                    | method | Generate signed ImageKit.io URLs                                           |
| urlEndpoint            | string | The ImageKit.io endpoint that was passed into the ImageKitProvider         |
| publicKey              | string | The ImageKit.io public key that was passed into the ImageKitProvider       |
| authenticationEndpoint | string | The authentication endpoint that was passed into the ImageKitProvider      |
| imageKitClient         | object | Raw ImageKit SDK client                                                    |
