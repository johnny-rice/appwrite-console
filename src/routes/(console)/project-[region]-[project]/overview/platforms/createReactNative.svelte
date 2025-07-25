<script lang="ts">
    import { Wizard } from '$lib/layout';
    import { invalidate } from '$app/navigation';
    import { createPlatform } from './wizard/store';
    import { Dependencies } from '$lib/constants';
    import {
        Card as Pink2Card,
        Code,
        Layout,
        Icon,
        Typography,
        Fieldset,
        InlineCode,
        Tooltip
    } from '@appwrite.io/pink-svelte';
    import { Button, Form, InputText } from '$lib/elements/forms';
    import { IconReact, IconAppwrite, IconInfo } from '@appwrite.io/pink-icons-svelte';
    import { Card } from '$lib/components';
    import { page } from '$app/state';
    import { onMount } from 'svelte';
    import { sdk } from '$lib/stores/sdk';
    import { Submit, trackError, trackEvent } from '$lib/actions/analytics';
    import { addNotification } from '$lib/stores/notifications';
    import { fade } from 'svelte/transition';
    import ConnectionLine from './components/ConnectionLine.svelte';
    import OnboardingPlatformCard from './components/OnboardingPlatformCard.svelte';
    import { PlatformType } from '@appwrite.io/console';

    let showExitModal = false;
    let isPlatformCreated = false;
    let isCreatingPlatform = false;
    let connectionSuccessful = false;
    const projectId = page.params.project;

    const gitCloneCode =
        '\ngit clone https://github.com/appwrite/starter-for-react-native\ncd starter-for-react-native\n';

    const updateConfigCode = `EXPO_PUBLIC_APPWRITE_PROJECT_ID=${projectId}
EXPO_PUBLIC_APPWRITE_ENDPOINT=${sdk.forProject(page.params.region, page.params.project).client.config.endpoint}`;

    export let platform: PlatformType = PlatformType.Reactnativeandroid;

    let platforms: { [key: string]: PlatformType } = {
        Android: PlatformType.Reactnativeandroid,
        iOS: PlatformType.Reactnativeios
    };

    const placeholder: Partial<
        Record<
            PlatformType,
            {
                name: string;
                hostname: string;
                tooltip: string;
            }
        >
    > = {
        [PlatformType.Reactnativeandroid]: {
            name: 'My Android App',
            hostname: 'com.company.appname',
            tooltip:
                'Your package name is generally the applicationId in your app-level build.gradle file.'
        },
        [PlatformType.Reactnativeios]: {
            name: 'My iOS App',
            hostname: 'com.company.appname',
            tooltip:
                "You can find your Bundle Identifier in the General tab for your app's primary target in Xcode."
        }
    };

    const hostname: Partial<Record<PlatformType, string>> = {
        [PlatformType.Reactnativeandroid]: 'Package name',
        [PlatformType.Reactnativeios]: 'Bundle ID'
    };

    async function createReactNativePlatform() {
        try {
            isCreatingPlatform = true;
            await sdk.forConsole.projects.createPlatform(
                projectId,
                platform,
                $createPlatform.name,
                $createPlatform.key || undefined,
                undefined,
                undefined
            );

            isPlatformCreated = true;
            trackEvent(Submit.PlatformCreate, {
                type: platform
            });

            invalidate(Dependencies.PROJECT);
            invalidate(Dependencies.PLATFORMS);
        } catch (error) {
            trackError(error, Submit.PlatformCreate);
            addNotification({
                type: 'error',
                message: error.message
            });
        } finally {
            isCreatingPlatform = false;
        }
    }

    async function resetPlatformStore() {
        createPlatform.reset();
    }

    onMount(() => {
        const unsubscribe = sdk.forConsole.client.subscribe('console', (response) => {
            if (response.events.includes(`projects.${projectId}.ping`)) {
                connectionSuccessful = true;
                invalidate(Dependencies.ORGANIZATION);
                invalidate(Dependencies.PROJECT);
                unsubscribe();
            }
        });

        return () => {
            unsubscribe();
            resetPlatformStore();
        };
    });
</script>

<Wizard title="Add React Native platform" bind:showExitModal confirmExit={!isPlatformCreated}>
    <Layout.Stack gap="xxl">
        <Form onSubmit={createReactNativePlatform}>
            <Layout.Stack gap="xxl">
                <!-- Step One -->
                <Layout.Stack gap="l" direction="row">
                    {#each Object.entries(platforms) as [key, value]}
                        <Pink2Card.Selector
                            {value}
                            id={key}
                            title={key}
                            imageRadius="s"
                            name="framework"
                            bind:group={platform}
                            disabled={isCreatingPlatform || isPlatformCreated} />
                    {/each}
                </Layout.Stack>

                <!-- Step Two -->
                {#if !isPlatformCreated}
                    <Fieldset legend="Details">
                        <Layout.Stack gap="l" alignItems="flex-end">
                            <Layout.Stack gap="s">
                                <InputText
                                    id="name"
                                    label="Name"
                                    placeholder={placeholder[platform].name}
                                    required
                                    bind:value={$createPlatform.name} />

                                <!-- Tooltips on InputText don't work as of now -->
                                <InputText
                                    id="hostname"
                                    label={hostname[platform]}
                                    placeholder={placeholder[platform].hostname}
                                    required
                                    bind:value={$createPlatform.key}>
                                    <Tooltip slot="info" maxWidth="15rem">
                                        <Icon icon={IconInfo} size="s" />
                                        <Typography.Caption variant="400" slot="tooltip">
                                            {placeholder[platform].tooltip}
                                        </Typography.Caption>
                                    </Tooltip>
                                </InputText>
                            </Layout.Stack>

                            <Button
                                fullWidthMobile
                                size="s"
                                submit
                                forceShowLoader
                                submissionLoader={isCreatingPlatform}
                                disabled={!platform ||
                                    !$createPlatform.name ||
                                    !$createPlatform.key ||
                                    isCreatingPlatform}>
                                Create platform
                            </Button>
                        </Layout.Stack>
                    </Fieldset>
                {:else}
                    <Layout.Stack gap="xxl">
                        <Card padding="s" radius="s">
                            <Layout.Stack
                                direction="row"
                                justifyContent="space-between"
                                alignItems="center"
                                gap="xs">
                                <Layout.Stack direction="row" alignItems="center" gap="s">
                                    <Icon size="m" icon={IconReact} />
                                    <Typography.Text
                                        variant="m-400"
                                        color="--fgcolor-neutral-primary">
                                        {$createPlatform.name} ({$createPlatform.key})
                                    </Typography.Text>
                                </Layout.Stack>
                            </Layout.Stack>
                        </Card>
                    </Layout.Stack>
                {/if}
            </Layout.Stack>
        </Form>

        <!-- Step Three -->
        {#if isPlatformCreated}
            <Fieldset legend="Clone starter">
                <Layout.Stack gap="l">
                    <Typography.Text variant="m-500">
                        1. If you're starting a new project, you can clone our starter kit from
                        GitHub using the terminal or VSCode.
                    </Typography.Text>

                    <!-- Temporary fix: Remove this div once Code splitting issue with stack spacing is resolved -->
                    <div class="pink2-code-margin-fix">
                        <Code lang="bash" lineNumbers code={gitCloneCode} />
                    </div>

                    <Typography.Text variant="m-500"
                        >2. Add your Appwrite credentials to <InlineCode
                            size="s"
                            code=".env.example" /> then rename it to <InlineCode
                            size="s"
                            code=".env" /> if needed.</Typography.Text>

                    <!-- Temporary fix: Remove this div once Code splitting issue with stack spacing is resolved -->
                    <div class="pink2-code-margin-fix">
                        <Code lang="dotenv" lineNumbers code={updateConfigCode} />
                    </div>

                    <Typography.Text variant="m-500"
                        >3. Run the app on a connected device or simulator, then click the <InlineCode
                            size="s"
                            code="Send a ping" /> button to verify the setup.</Typography.Text>
                </Layout.Stack>
            </Fieldset>
        {/if}
    </Layout.Stack>
    <svelte:fragment slot="aside">
        <Card padding="l" class="responsive-padding">
            <Layout.Stack gap="xxl">
                <Layout.Stack direction="row" justifyContent="center" gap="none">
                    <OnboardingPlatformCard iconSize={2.526} iconColor="#61DBFB" icon={IconReact} />

                    <ConnectionLine status={connectionSuccessful} />

                    <OnboardingPlatformCard
                        iconSize={2.526}
                        iconColor="#FD366E"
                        icon={IconAppwrite} />
                </Layout.Stack>

                {#if isPlatformCreated}
                    <Layout.Stack
                        direction="row"
                        justifyContent="center"
                        alignItems="center"
                        gap="l">
                        {#if !connectionSuccessful}
                            <Typography.Text variant="m-400"
                                >Waiting for connection...</Typography.Text>
                        {:else}
                            <!-- cannot apply fade on components -->
                            <div
                                in:fade={{ duration: 2500 }}
                                class="u-flex u-flex-vertical u-cross-center u-gap-8">
                                <Typography.Title size="m">Congratulations!</Typography.Title>

                                <Typography.Text variant="m-400"
                                    >You connected your app successfully.</Typography.Text>
                            </div>
                        {/if}
                    </Layout.Stack>
                {/if}
            </Layout.Stack>
        </Card>
    </svelte:fragment>

    <svelte:fragment slot="footer">
        {#if isPlatformCreated}
            <Button
                size="s"
                fullWidthMobile
                secondary
                disabled={isCreatingPlatform}
                href={location.pathname}>
                Go to dashboard
            </Button>
        {/if}
    </svelte:fragment>
</Wizard>

<style lang="scss">
    :global(.pink2-code-margin-fix pre) {
        margin: revert;
    }

    :global(.responsive-padding) {
        @media (max-width: 768px) {
            padding: 16px;
        }
    }
</style>
