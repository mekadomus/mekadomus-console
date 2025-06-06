<script lang="ts">
  import { SvelteMap } from 'svelte/reactivity';
  import { getContext } from 'svelte';
  import { goto } from '$app/navigation';
  import { page } from '$app/stores';

  import type { CreateFluidMeterInput } from '@api/FluidMeter';
  import type { Message } from '@api/Message';

  import MdCenteredContainer from '@components/MdCenteredContainer.svelte';
  import { createFluidMeter } from '@api/FluidMeter';
  import { MessageType } from '@api/Message';
  import { t, locale, loadTranslations } from '@utils/Translations';

  let lang = $state($page.params.lang);
  locale.set($page.params.lang);
  $effect(() => {
    lang = $page.params.lang;
    locale.set(lang);
    loadTranslations(lang, 'meter');
  });

  const globalMessages: SvelteMap<string, Message> = getContext('globalMessages');

  async function create() {
    const form = document.getElementById('new-meter-form') as HTMLFormElement;
    const name = document.getElementById('name') as HTMLFormElement;

    if (!form.checkValidity()) {
      form.reportValidity();
      return;
    }

    const data: CreateFluidMeterInput = {
      name: name.value
    };
    const res = await createFluidMeter(data);
    if ('owner_id' in res) {
      goto(`/${$page.params.lang}/meter/${res.id}/created?name=${encodeURI(res.name)}`);
    } else {
      let message: Message = {
        type: MessageType.Error,
        text: $t('meter.creation-error')
      };
      globalMessages.set('new-meter-error', message);
    }
  }
</script>

<MdCenteredContainer>
  <h1>{$t('meter.create-meter')}</h1>
  <form action="#" method="POST" id="new-meter-form">
    <div class="form-group">
      <label for="name">{$t('meter.name')}</label>
      <input type="name" id="name" name="name" required />
    </div>

    <button class="button" type="button" onclick={() => create()}
      >{$t('meter.create-button')}</button
    >
  </form>
</MdCenteredContainer>

<style>
  h1 {
    margin: 0;
    padding-bottom: 1rem;
  }

  button {
    margin-top: 1rem;
  }
</style>
