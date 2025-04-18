import { mount } from '@vue/test-utils';
import LoginView from '@/views/LoginView.vue';
import Server from '@/Server';

jest.mock('@/Server', () => {
  return jest.fn(() => ({
    post: jest.fn(() => Promise.resolve({ data: { access_token: 'mock-token' } }))
  }));
});

describe('LoginView.vue', () => {
  it('renders the login form', () => {
    const wrapper = mount(LoginView);
    expect(wrapper.find('form').exists()).toBe(true);
    expect(wrapper.find('#email').exists()).toBe(true);
    expect(wrapper.find('#password').exists()).toBe(true);
    expect(wrapper.find('button[type="submit"]').exists()).toBe(true);
  });

  it('shows an error message when login fails', async () => {
    Server().post.mockRejectedValueOnce({
      response: { data: { error: 'Invalid credentials' } }
    });

    const wrapper = mount(LoginView);
    await wrapper.setData({ email: 'test@example.com', password: 'wrongpassword' });
    await wrapper.find('form').trigger('submit.prevent');

    await wrapper.vm.$nextTick(); // attendre la mise à jour du DOM
    expect(wrapper.vm.status).toBe('failed');
    expect(wrapper.find('.error-message').text()).toBe('Invalid credentials');
  });

  it('redirects to home on successful login', async () => {
    const pushMock = jest.fn();
    const wrapper = mount(LoginView, {
      mocks: {
        $router: { push: pushMock }
      }
    });

    await wrapper.setData({ email: 'test@example.com', password: 'correctpassword' });
    await wrapper.find('form').trigger('submit.prevent');

    expect(localStorage.getItem('token')).toBe('mock-token');
    expect(pushMock).toHaveBeenCalledWith('/');
  });
});